export PROJECT=syw-cdw-ti-dev
export WORKERS=0
export REGION=us-central1 
export ZONE=us-central1-c
export BUCKET_NAME=gcs_dataproc_presto
export INIT_ACTION=gs://dataproc-initialization-actions/presto/presto.sh

gcloud dataproc clusters create presto-cluster \
     --project=${PROJECT} \
     --zone=${ZONE} \
     --num-workers=${WORKERS} \
     --scopes=cloud-platform \
     --initialization-actions=${INIT_ACTION}


     gcloud dataproc jobs submit hive \
    --cluster presto-cluster \
    --execute "
        CREATE SCHEMA IF NOT EXISTS ExternalHive;
        DROP TABLE IF EXISTS ExternalHive.Nseg;
        CREATE EXTERNAL TABLE IF NOT EXISTS ExternalHive.Nseg (
        TrxnDt date COMMENT 'Transaction Date',
        MemberID string COMMENT 'Member ID',
        SegmentID string COMMENT 'Segment ID'
        )
        COMMENT 'External Hive Table, Raw data stored in GCS Bucket : Table- Nseg'
        ROW FORMAT SERDE
            'org.apache.hadoop.hive.serde2.OpenCSVSerde'
        WITH SERDEPROPERTIES (
            'separatorChar' = '|',
            'quoteChar' = '\"',
            'escapeChar' = '\\\\')
        STORED AS INPUTFORMAT
            'org.apache.hadoop.mapred.TextInputFormat'
        OUTPUTFORMAT
            'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
        LOCATION
            'gs://gcs_sachinkoli/nseg/'
        TBLPROPERTIES (
            'serialization.null.format' = '',
            'skip.header.line.count' = '1');
    "
    
    
    gcloud dataproc jobs submit hive \
     --cluster presto-cluster \
     --execute " select count(*) from ExternalHive.Nseg"

    gcloud dataproc jobs submit hive \
      --cluster presto-cluster \
      --execute "use ExternalHive; show tables;" 
    
    Run on local machine
    
    gcloud compute ssh presto-cluster-m \
    --project='syw-cdw-ti-dev' \
    --zone='us-central1-c' \
    -- -D 1080 -N
    
    
    
    ./presto-cli \
    --server presto-cluster-m:8080 \
    --socks-proxy localhost:1080 \
    --catalog hive \
    --schema default

    
    ./presto-cli \
    --server localhost:8080 \
    --catalog hive \
    --schema default


https://github.com/airbnb/airpal
    
    
    CREATE USER 'root'@'localhost' IDENTIFIED BY 'root123';
    
    update user set password=PASSWORD("root123") where User='root';
    
    GRANT ALL ON airpal.* TO root@localhost IDENTIFIED BY 'root123';
    GRANT ALL ON airpal TO root@localhost;
FLUSH privileges;

mysqlcheck -u root -p --all-databases --check-upgrade --auto-repair
mysql_upgrade -u root -p


CREATE TABLE externalhive.request_logs (
  request_time timestamp,
  url varchar,
  ip varchar,
  user_agent varchar
)
WITH (
  format = 'TEXTFILE',
  external_location = 'gs://gcs_sachinkoli/request_logs/'
)


airpal
    java -server \
         -Duser.timezone=UTC \
         -cp build/libs/airpal-*-all.jar com.airbnb.airpal.AirpalApplication server reference.yml
     
hue
     /usr/lib/hue/build/env/bin/supervisor -p /var/run/hue/supervisor.pid -d -l /var/log/hue
