## Rachel's guide for automatic uploading and downloading of files using the 0G storage client



-----------------------------------------------------------------


## 1. Create a Directory for Downloads
    mkdir -p /root/0g-storage-client/downloadfile
    

## 2. Create the Go File for Calculating Root Hash

**1. Create the `root_hash.go` file in 0g-storage-client directory:**
        
    cd /root/0g-storage-client
    nano root_hash.go

**2. Paste the following code into `root_hash.go`:**

    go
    package main

    import (
        "fmt"
        "os"
        "github.com/0glabs/0g-storage-client/core"
        "github.com/pkg/errors"
    )

    func calculateMerkleRoot(filePath string) (string, error) {
        // Open the file
        file, err := core.Open(filePath)
        if err != nil {
            return "", errors.WithMessage(err, "Failed to open file")
        }
        defer file.Close()

        // Calculate the Merkle tree for the file
        tree, err := core.MerkleTree(file)
        if err != nil {
            return "", errors.WithMessage(err, "Failed to create merkle tree")
        }

        // Get the Merkle root hash
        merkleRoot := tree.Root()

        return merkleRoot.Hex(), nil
    }

    func main() {
        if len(os.Args) < 2 {
            fmt.Println("Usage: root_hash <file_path>")
            os.Exit(1)
        }

        filePath := os.Args[1]

        merkleRoot, err := calculateMerkleRoot(filePath)
        if err != nil {
            fmt.Printf("Failed to calculate merkle root: %v\n", err)
            os.Exit(1)
        }

        fmt.Println(merkleRoot)
    }
    
**4. Save the file and exit the editor:**
   For `nano`, press `Ctrl + X`, then `Y` to confirm saving, and `Enter` to exit.

## 3. Build the Go File
Run the following command to build the Go file:

    go build -o root_hash root_hash.go

## 4. Create the Upload Script

**1. Create the `upload.sh` file:**

    nano /root/0g-storage-client/upload.sh

**2. Paste the following code into `upload.sh`:**
 **!REMEMBER** Change `NODE_URL`,`UPLOAD_URL`, `KEY` by your your storage node url (http://storage_node_ip:5678) your json rpc endpoint (http://validator_node_ip:8545) and your privatekey

    #!/bin/bash

    # Create the downloadfile directory if it doesn't exist
    mkdir -p /root/0g-storage-client/downloadfile

    # Path to the log file
    LOG_FILE="/root/0g-storage-client/upload_log_$(date +"%Y%m%d").log"
    DOWNLOAD_LOG_FILE="/root/0g-storage-client/download_log_$(date +"%Y%m%d").log"

    # Initialize the number of successful uploads and downloads
    UPLOAD_COUNT=0
    DOWNLOAD_COUNT=0

    for i in $(seq 1 1000000)
    do
        FILE="file_$(date +"%Y%m%d_%H%M%S")"

        NODE_URL="http://<your_storage_ip>:5678/"
        UPLOAD_URL="http://<your_validator_ip>:8545/"
        CONTRACT="0x8873cc79c5b3b5666535C825205C9a128B1D75F1"
        KEY="your_privatekey"

        # Generate a new file with size 1MB (1,048,576 bytes)
        /root/0g-storage-client/0g-storage-client gen --size 1048576 --file "$FILE"

        # Calculate Merkle root hash and save to variable $AA
        AA=$(./root_hash "$FILE")

        # Upload the file
        /root/0g-storage-client/0g-storage-client upload \
        --url "$UPLOAD_URL" \
        --contract "$CONTRACT" \
        --key "$KEY" \
        --node "$NODE_URL" \
        --file "$FILE"

        # Check if the file was uploaded successfully
        if [ $? -eq 0 ]; then
            UPLOAD_COUNT=$((UPLOAD_COUNT + 1))
            echo "$(date +"%Y-%m-%d %H:%M:%S") - Successfully uploaded file: $FILE" >> "$LOG_FILE"
        else
            echo "$(date +"%Y-%m-%d %H:%M:%S") - Failed to upload file: $FILE" >> "$LOG_FILE"
        fi
        echo -e "\033[1;33m Uploaded file $FILE with Merkle root $AA\n\033[0m"

        # Wait 1 second before the next upload
        sleep 1

        # Download the file using the Merkle root $AA
        OUTPUT_FILE="/root/0g-storage-client/downloadfile/downloaded_$FILE"
        /root/0g-storage-client/0g-storage-client download \
        --node "$NODE_URL" \
        --root "$AA" \
        --file "$OUTPUT_FILE"

        # Check if the file was downloaded successfully
        if [ $? -eq 0 ]; then
            DOWNLOAD_COUNT=$((DOWNLOAD_COUNT + 1))
            echo "$(date +"%Y-%m-%d %H:%M:%S") - Successfully downloaded file: $OUTPUT_FILE" >> "$DOWNLOAD_LOG_FILE"
        else
            echo "$(date +"%Y-%m-%d %H:%M:%S") - Failed to download file: $OUTPUT_FILE" >> "$DOWNLOAD_LOG_FILE"
        fi

        echo -e "\033[1;33m Successfully downloaded $OUTPUT_FILE\n\033[0m"

        # After uploading and downloading, delete the files to keep the system clean
        rm "$FILE"
        rm "$OUTPUT_FILE"

        # Wait 1 second before the next operation
        sleep 1
    done

    # Record the total number of successful uploads and downloads
    echo "$(date +"%Y-%m-%d %H:%M:%S") - Total files successfully uploaded: $UPLOAD_COUNT" >> "$LOG_FILE"
    echo "$(date +"%Y-%m-%d %H:%M:%S") - Total files successfully downloaded: $DOWNLOAD_COUNT" >> "$DOWNLOAD_LOG_FILE"

**3. Save the file and exit the editor:**
For `nano`, press `Ctrl + X`, then `Y` to confirm saving, and `Enter` to exit.

## 5. Make the Script Executable

**1. Run the following command to make the script executable:**

    chmod a+x /root/0g-storage-client/upload.sh

## 6: Schedule the Script to Run at Midnight Using Cron

**1. Open the crontab editor:**

    crontab -e

**2. Add the following line to schedule the script to run at midnight every day:**

    0 0 * * * /root/0g-storage-client/upload.sh

**3. Save the file and exit the editor:**
For `nano`, press `Ctrl + X`, then `Y` to confirm saving, and `Enter` to exit.

-----------------------------------------------------------------

## Helpful tool by Racchel

 **Run the script manually to start the upload and download process:**

    ./upload.sh

 **To check the number of successful uploads in a day:**

    grep "Successfully uploaded" /root/0g-storage-client/upload_log_$(date +"%Y%m%d").log | wc -l

 **To check the number of successful downloads in a day:**

    grep "Successfully downloaded" /root/0g-storage-client/download_log_$(date +"%Y%m%d").log | wc -l

