## Tạo data registry repo:
1. `git init`
2. `dvc init`

### 3a. Nexus repo
* `dvc remote add -d <remote-name> <repo-url>` \
(vd `dvc remote add -d nexus-repo https://nexus.htsc.vn/repository/htsc-raw-release-hosted/Team-AI/dvc`)
* `dvc remote modify <remote-name> auth basic`
* `dvc remote modify <remote-name> method PUT`
* `dvc remote modify --local <remote-name> user <user>` \
(option `--local` để git không track thông tin nhạy cảm)
* `dvc remote modify --local <remote-name> password <password>`
### 3b. Minio (s3 compatible storage)
* `dvc remote add -d <remote-name> s3://<bucket-name>`
* `dvc remote modify <remote-name> endpointurl <minio_url>` \
(vd `http://localhost:9000`)
* `dvc remote modify --local <remote-name> credentialpath <credential_file_path>` \
VD credential file:
```
[default]
aws_access_key_id = ...
aws_secret_access_key = ...
```

4. `dvc add <dir/file>`: track file với dvc (generate ra file `.dvc` tương ứng để track bằng git), lưu file vào cache
5. `git add <dir/file>.dvc; git commit -m "..."; git push ...` \
`(Optional) dvc status --remote <remote-name>`: so sánh cache vs remote
6. `dvc push`: đẩy data từ cache vào remote storage