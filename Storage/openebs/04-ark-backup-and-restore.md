## Backup and Restore

### Heptio Ark / Velero

find the latest release from
[https://github.com/heptio/velero/releases]

```wget <<RELEASE-TARBALL-NAME>.tar.gz>```

``` tar -xvf <RELEASE-TARBALL-NAME>.tar.gz -C velero```

```sudo cp velero/velero /usr/bin/velero```

### install minio in cluster

```cd velero```
```kubectl apply -f minio/```

Now minio client will be installed in the velero namespace in your kubernetes cluster. 

NOTE: Ark/velero supports GCP bucket integration and S3 Bucket integration. However in on prem since we don't have both, we have to resort to the minio client which emulates an AWS S3 bucket on prem.

You have to expose the minio service either by spcifying it as a NodePort or adding an ingress to it.

```kubectl edit svc minio -n velero```

35.238.13.182 

### Connect minio and velero

``` vi credentials-velero```

``` 
[default]
 aws_access_key_id = minio
 aws_secret_access_key = minio123
 ```

 ```
 velero install \
     --provider aws \
     --bucket velero \
     --secret-file ./credentials-velero \
     --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=<minio exposed url>:9000 \
```

Velero is installed! ⛵ Use 'kubectl logs deployment/velero -n velero' to view the status.

Visit 
[https://heptio.github.io/velero/master/get-started] to see how to do backups and restore