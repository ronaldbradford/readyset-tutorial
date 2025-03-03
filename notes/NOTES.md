docker run -d \
--name=readyset \
--publish=3307:3307 \
--platform=linux/amd64 \
--volume='readyset:/state' \
--pull=always \
-e DEPLOYMENT_ENV=quickstart_github \
public.ecr.aws/readyset/readyset:beta-2022-12-15 \
--standalone \
--deployment='github-mysql' \
--database-type=mysql \
--upstream-db-url=mysql://root:readyset@172.17.0.1:3306/testdb \
--address=0.0.0.0:3307 \
--username='root' \
--password='readyset' \
--query-caching='explicit' \
--db-dir='/state'

