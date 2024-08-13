## Rachel's guide for running uploading and downloading of 0G storage client in the background



-----------------------------------------------------------------

## 0. Build Storage CLI with source code
 ```bash
git clone https://github.com/0glabs/0g-storage-client.git
cd 0g-storage-client
go build
 ```


## 1. Create a Directory for Downloads
    
    mkdir -p /root/0g-storage-client/downloadfile
    

## 2. Create the Go File for Merkle Root Calculation

**1. Create the `root_hash.go` file in 0g-storage-client directory:**
        
    cd /root/0g-storage-client
    nano root_hash.go

**2. Paste the following code into `root_hash.go`:**
```
package main

import (
    "fmt"
    "os"
    "github.com/0glabs/0g-storage-client/core"
    "github.com/pkg/errors"
)

func calculateMerkleRoot(filePath string) (string, error) {
    file, err := core.Open(filePath)
    if err != nil {
        return "", errors.WithMessage(err, "Failed to open file")
    }
    defer file.Close()

    tree, err := core.MerkleTree(file)
    if err != nil {
        return "", errors.WithMessage(err, "Failed to create merkle tree")
    }

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
```

**3. Save the file and exit the editor:**
   For `nano`, press `Ctrl + X`, then `Y` to confirm saving, and `Enter` to exit.

## 3. Build the Go File
Run the following command to build the Go file:

    go build -o root_hash root_hash.go

If you encountered the error like this picture, This is because there is no go.mod file in your directory, which is required for dependency management in Go.
![image](https://github.com/user-attachments/assets/f7963bda-2415-46f7-b4e0-815a0e4a141e)

To resolve this issue, you need to initialize a Go module in your project directory and then add the necessary dependencies. Here’s how you can do it:

*Initialize the Go Module*

    cd /root/0g-storage-client
    go mod init github.com/rachel/0g-storage-client

*Add Required Dependencies*

    go get github.com/0glabs/0g-storage-client/core
    go get github.com/pkg/errors

## 4. Ensure root_hash Has Execution Permissions
Ensure that the root_hash script is executable.

    chmod a+x /root/0g-storage-client/root_hash

## 4. Create the Upload Script

**1. Create the `upload.sh` file:**

    nano /root/0g-storage-client/download_upload.sh

**2. Paste the following code into `upload.sh`:**
 
 **!REMEMBER** Change this variables
 
 **`NODE_URL`**    :your your storage node url (http://storage_node_ip:5678)
 
 **`UPLOAD_URL`**  :your json rpc endpoint (http://validator_node_ip:8545)
 
 **`KEY`**         :your privatekey
```
#!/bin/bash

# Create downloadfile directory if it doesn't exist
mkdir -p /root/0g-storage-client/downloadfile

# Path to the log file
LOG_FILE="/root/0g-storage-client/upload_log_$(date +"%Y%m%d").log"

# Initialize counters for successful uploads and downloads
UPLOAD_COUNT=0
DOWNLOAD_COUNT=0

for i in $(seq 1 100000)
do
    FILE="/root/0g-storage-client/file_$(date +"%Y%m%d_%H%M%S")"

    NODE_URL="http://<your_storage_ip>:5678/"
    UPLOAD_URL="http://<your_validator_ip>:8545/"
    CONTRACT="0xbD2C3F0E65eDF5582141C35969d66e34629cC768"
    KEY="your_privatekey"

    # Generate a new file
    /root/0g-storage-client/0g-storage-client gen --size 102400 --file "$FILE"

    # Calculate Merkle root hash and save it to variable $AA
    AA=$(/root/0g-storage-client/root_hash "$FILE")

    # Log Merkle root
    echo "Generated Merkle root: $AA for file: $FILE" >> "$LOG_FILE"

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

    sleep 1

    # Download the file using Merkle root $AA
    OUTPUT_FILE="/root/0g-storage-client/downloadfile/downloaded_$(basename $FILE)"
    /root/0g-storage-client/0g-storage-client download \
    --node "$NODE_URL" \
    --root "$AA" \
    --file "$OUTPUT_FILE"

    # Check if the file was downloaded successfully
    if [ $? -eq 0 ]; then
        DOWNLOAD_COUNT=$((DOWNLOAD_COUNT + 1))
        echo "$(date +"%Y-%m-%d %H:%M:%S") - Successfully downloaded file: $OUTPUT_FILE" >> "$LOG_FILE"
    else
        echo "$(date +"%Y-%m-%d %H:%M:%S") - Failed to download file: $OUTPUT_FILE" >> "$LOG_FILE"
    fi

    # After uploading and downloading, delete the files to keep the system clean
    rm "$FILE"
    rm "$OUTPUT_FILE"
    sleep 1
done

# Record the total number of successful uploads and downloads
echo "$(date +"%Y-%m-%d %H:%M:%S") - Total files successfully uploaded: $UPLOAD_COUNT" >> "$LOG_FILE"
echo "$(date +"%Y-%m-%d %H:%M:%S") - Total files successfully downloaded: $DOWNLOAD_COUNT" >> "$LOG_FILE"
```

**3. Save the file and exit the editor:**
For `nano`, press `Ctrl + X`, then `Y` to confirm saving, and `Enter` to exit.

## 5. Make the Script Executable
Run the following command to make the script executable:

    chmod a+x /root/0g-storage-client/download_upload.sh

## 6. Run the Script in the Background
Run the script in the background and redirect the output to a log file.

    nohup /root/0g-storage-client/download_upload.sh > /root/0g-storage-client/background_run.log 2>&1 &

-----------------------------------------------------------------

## Helpful tool by Racchel
 
 **Check the Log File**
 If you want more detailed output, you can periodically check the background_run.log file for any runtime errors or issues.
 
    tail -f /root/0g-storage-client/background_run.log

You can see all the log file (/root/0g-storage-client/background_run.log) for any errors or issues. 

    cat /root/0g-storage-client/background_run.log

**Count Successful Uploads and Downloads**
Change `20240813` to the date you start run client.
For example, i start run client from 13/08/2024

      grep -c "Successfully uploaded file" /root/0g-storage-client/upload_log_20240813.log

      grep -c "Successfully downloaded file" /root/0g-storage-client/upload_log_20240813.log







   
