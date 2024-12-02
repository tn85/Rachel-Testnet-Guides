
## 0. Build Storage CLI with source code
 ```bash
git clone https://github.com/0glabs/0g-storage-client.git
cd 0g-storage-client
git tag -d v0.6.1
git fetch --all --tags
git checkout 88f563d81a60208a44ed7662a240e307277f7965
git submodule update --init
go build
 ```

## Bước 1: Cập nhật Script upload.sh ##
1. Mở file script `upload.sh`:
```
nano /root/0g-storage-client/upload.sh
```
2. Cập nhật nội dung script:
```
#!/bin/bash

# Path to the log file
LOG_FILE="/root/0g-storage-client/upload_log_$(date +"%Y%m%d").log"

# Initialize the number of successful uploads
UPLOAD_COUNT=0

for i in $(seq 1 20)  # Set the loop count to 20
do
    FILE="file_$(date +"%Y%m%d_%H%M%S")"

    UPLOAD_URL="http://IP_NODE_VALIDATOR:8545/"
    INDEXER="https://indexer-storage-testnet-standard.0g.ai"
    KEY="YOUR_PRIVATE_KEY"

    # Generate a new file
    /root/0g-storage-client/0g-storage-client gen --size 102400 --file "./$FILE"

    # Upload the file
    /root/0g-storage-client/0g-storage-client upload \
    --url "$UPLOAD_URL" \
    --key "$KEY" \
    --indexer "$INDEXER" \
    --file "./$FILE"
    --finality-required true

    # Check if the file was uploaded successfully
    if [ $? -eq 0 ]; then
        UPLOAD_COUNT=$((UPLOAD_COUNT + 1))
        echo "$(date +"%Y-%m-%d %H:%M:%S") - Successfully uploaded file: $FILE" >> "$LOG_FILE"
    else
        echo "$(date +"%Y-%m-%d %H:%M:%S") - Failed to upload file: $FILE" >> "$LOG_FILE"
    fi

    # After uploading, delete the file to keep the system clean
    rm "./$FILE"

    # Wait 5 seconds before uploading the next file (optional, to reduce load)
    sleep 5
done

# Record the total number of successful uploads
echo "$(date +"%Y-%m-%d %H:%M:%S") - Total files successfully uploaded: $UPLOAD_COUNT" >> "$LOG_FILE"
```
3.Lưu file và thoát khỏi trình chỉnh sửa: `Ctrl + X` ->  `Y` -> Enter
4. Đảm bảo script có quyền
```
chmod a+x /root/0g-storage-client/upload.sh
```

## Bước 2: Thiết lập lịch trình  tự động up script ##
1. Mở file crontab:
```
crontab -e
```
3. Chọn trình soạn thảo mong muốn: 
(Chọn /bin/nano ) Bấm `1` -> `Enter`
4. Thiết lập file crontab để chạy script vào mỗi giờ hàng ngày:
(Xoá hết chữ  trước đó và nhập lệnh sau)
```
0 * * * * /root/0g-storage-client/upload.sh
```
4.Lưu file và thoát khỏi trình chỉnh sửa: `Ctrl + X `->  `Y` -> `Enter`
## Bước 3: Kiểm tra crontab (để chắc chắn đã lên lịch thành công) ##
Nhập lệnh 
```
crontab -l
```
Nếu thấy dòng lênh sau thì thành công 
```
0 * * * * /root/0g-storage-client/upload.sh
```
## Bước 4: Kiểm tra số lần tải lên thành công trong ngày ##
- Kiểm tra SỐ LẦN tải lên thành công: 
```
grep "Successfully uploaded" /root/0g-storage-client/upload_log_$(date +"%Y%m%d").log | wc -l
```
- Kiểm tra chi tiết nhật kí upload file thành công:
```
grep "Successfully uploaded" /root/0g-storage-client/upload_log_$(date +"%Y%m%d").log
```
Chạy thủ công:
```
/root/0g-storage-client/upload.sh
```



   
