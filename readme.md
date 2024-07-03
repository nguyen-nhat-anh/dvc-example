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

## Update data registry repo:
4. `dvc add <dir/file>`: track file với dvc (generate ra file `.dvc` tương ứng để track bằng git), lưu file vào cache
5. `git add <dir/file>.dvc; git commit -m "..."; git push ...` \
`(Optional) dvc status --remote <remote-name>`: so sánh cache vs remote
6. `dvc push`: đẩy data từ cache vào remote storage

## Pull data từ data registry:
1. `git clone <data_registry_repo>`
### 2a. Nexus repo
* `dvc remote modify --local <remote-name> user <user>`
* `dvc remote modify --local <remote-name> password <password>`
### 2b. Minio
* `dvc remote modify --local <remote-name> credentialpath <credential_file_path>`
3. `dvc fetch`: download remote về cache
4. `dvc checkout`: cache -> workspace \
(`dvc pull` = `dvc fetch` + `dvc checkout`)

## Download file/folder thẳng từ registry:
### Nexus repo
1. Config credential: tạo một file config giống như `.dvc/config.local` lưu user và password. VD:
```
['remote "nexus-repo"']
    user = ...
    password = ...
```
2. `dvc get -o <output_path> --config <config_path> --rev <commit/tag/branch> <data_registry_repo> <path_in_repo>`
### Minio
1. Config credential: chẳng hạn export `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY`
2. `dvc get -o <output_path> --rev <commit/tag/branch> <data_registry_repo> <path_in_repo>`

## Notes
* `dvc add`: track file với dvc (generate ra file `.dvc` tương ứng để track bằng git) (`dvc add --no-commit`) + lưu file vào cache (`dvc commit`)
* `dvc fetch`: download data từ remote về cache
* `dvc checkout`: update workspace dựa vào file `.dvc` tương ứng (data lấy từ cache ra)
* `dvc pull` = `dvc fetch` + `dvc checkout`
* `dvc status`: so sánh workspace vs cache
    * `--remote <remote-name>`: so sánh cache vs remote
* `dvc push`: upload data từ cache (kiểm tra các file `.dvc` ở workspace, xác định các file nào trong cache chưa upload) lên remote storage 
    * `-a` (`--all-branches`): không chỉ kiểm tra `.dvc` ở workspace mà còn ở tất cả các git branch nữa
    * `-T` (`--all-tags`): không chỉ kiểm tra `.dvc` ở workspace mà còn ở tất cả các git tag nữa
    * `-A` (`--all-commits`): không chỉ kiểm tra `.dvc` ở workspace mà còn ở tất cả các git commit nữa
* `dvc install`: install git hooks để tự động hoá một số thao tác với `dvc`:
    * `post-checkout`: thực hiện `dvc checkout` sau `git checkout` để tự động update workspace
    * `pre-commit`: thực hiện `dvc status` trước `git commit` để tự động thông báo thay đổi trong workspace so với cache
    * `pre-push`: thực hiện `dvc push` trước `git push` để tự động upload các file được track với dvc lên remote storage