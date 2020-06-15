# Backing up Neo4j Containers

This directory contains files necessary for backing up Neo4j Docker containers
to google storage.

**This approach assumes you have Google Cloud credentials and wish to store your backups
on Google Cloud Storage**.  If this is not the case, you will need to adjust the backup
script for your desired cloud storage method, but the approach will work for any backup location.

### Create a service key secret to access cloud storage

First you want to create a kubernetes secret that contains the content of your account service key.  This key must have permissions to access the bucket and backup set that you're trying to restore. 

```
MY_SERVICE_ACCOUNT_KEY=$HOME/.google/my-service-key.json
kubectl create secret generic neo4j-service-key \
   --from-file=credentials.json=$MY_SERVICE_ACCOUNT_KEY
```

The backup container is going to take this kubernetes secret
(named `neo4j-service-key`) and is going to mount it as a file
inside of the backup container (`/auth/credentials.json`).  That
file will then be used to authenticate the storage client that we
need to upload the backupset to cloud storage when it's complete.

### Running a Backup

See backup.yaml for an example.   You must have first created a `neo4j-service-key`
secret in the same namespace as your Neo4j is running.

**Required environment variables**

* `NEO4J_ADDR` pointing to an address where your cluster is running, ideally the
discovery address.
* `BUCKET` where you want the backup copied to.  It should be `gs://bucketname`

**Optional environment variables**

* `BACKUP_NAME` - the name of the backupset as a file.  (Default: neo4j)
* `DATABASE` - the database to back up.  (Default: neo4j)

```
kubectl apply -f backup.yaml --namespace my-neo4j-namespace
```

The "credentials.json" file must be a base64-encoded version of a service key JSON that has permissions to write to the targeted google storage bucket.  The example provided is non-functional, and you must substitute your own.  To determine an appropriate value, perform the following:

- Create a service account with appropriate permissions to write to the google
storage bucket
- Save the key in JSON format to your local disk
- `cat my-key.json | base64`
- Use that resulting value in your `backup.yaml` file
- Finally, after adjusting parameters in backup.yaml, run `kubectl apply -f backup.yaml --namespace my-neo4j-deployed-namespace`