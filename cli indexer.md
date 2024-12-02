## Rachel's guide for running uploading of 0G storage client with indexer in the background



-----------------------------------------------------------------

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

## 4. Create the Upload Script

**1. Create the `upload.sh` file:**

    nano /root/0g-storage-client/upload.sh

**2. Paste the following code into `upload.sh`:**
 
 **!REMEMBER** Change this variables
 
 **`NODE_URL`**    :your your storage node url (http://storage_node_ip:5678)
 
 **`UPLOAD_URL`**  :your json rpc endpoint (http://validator_node_ip:8545)
 
 **`KEY`**         :your privatekey
```
#!/bin/bash

# Path to the log file
LOG_FILE="/root/0g-storage-client/upload_log_$(date +"%Y%m%d").log"

# Initialize counters for successful uploads and downloads
UPLOAD_COUNT=0

for i in $(seq 1 100000)
do
    FILE="/root/0g-storage-client/file_$(date +"%Y%m%d_%H%M%S")"

    NODE_URL="http://<your_storage_ip>:5678/"
    KEY="your_privatekey"
    INDEXER="https://indexer-storage-testnet-standard.0g.ai"

    # Generate a new file
    /root/0g-storage-client/0g-storage-client gen --size 102400 --file "$FILE"

    # Upload the file
    /root/0g-storage-client/0g-storage-client upload \
    --url "$UPLOAD_URL" \
    --key "$KEY" \
    --indexer "$INDEXER" \
    --file "$FILE" \
    --finality-required true

    # Check if the file was uploaded successfully
    if [ $? -eq 0 ]; then
        UPLOAD_COUNT=$((UPLOAD_COUNT + 1))
        echo "$(date +"%Y-%m-%d %H:%M:%S") - Successfully uploaded file: $FILE" >> "$LOG_FILE"
    else
        echo "$(date +"%Y-%m-%d %H:%M:%S") - Failed to upload file: $FILE" >> "$LOG_FILE"
    fi

    # After uploading and downloading, delete the files to keep the system clean
    rm "$FILE"
    sleep 3
done

# Record the total number of successful uploads and downloads
echo "$(date +"%Y-%m-%d %H:%M:%S") - Total files successfully uploaded: $UPLOAD_COUNT" >> "$LOG_FILE"
```

**3. Save the file and exit the editor:**
For `nano`, press `Ctrl + X`, then `Y` to confirm saving, and `Enter` to exit.

## 5. Make the Script Executable
Run the following command to make the script executable:

    chmod a+x /root/0g-storage-client/upload.sh

## 6. Run the Script in the Background
Run the script in the background and redirect the output to a log file.

    nohup /root/0g-storage-client/upload.sh > /root/0g-storage-client/background_run.log 2>&1 &

-----------------------------------------------------------------

## Helpful tool by Racchel
 
 **Check the Log File**
 If you want more detailed output, you can periodically check the background_run.log file for any runtime errors or issues.
 
    tail -f /root/0g-storage-client/background_run.log

You can see all the log file (/root/0g-storage-client/background_run.log) for any errors or issues. 

    cat /root/0g-storage-client/background_run.log

**Count Successful Uploads and Downloads**
Change `20240905` to the date you start run client.
For example, i start run client from 04/09/2024

      grep -c "Successfully uploaded file" /root/0g-storage-client/upload_log_20240904.log








   
