# ronin-snapshot

## Prerequisites
- Your free disk space has more than twice the size of the snapshot.
- Install the [zstd](https://github.com/facebook/zstd) on your machine.
- Install the [tmux](https://github.com/tmux/tmux/wiki/Installing) for long time process operation.


### Endpoint

- mainnet: [chaindata-20240124.tar.zst](https://pub-3cca138de6c349f8afe5f6635f9f6f81.r2.dev/data/chaindata-20240124.tar.zst) - md5: 185b331a21fad339fbdb2f6c27b913a0
- testnet: [testnet-chaindata-20240125.tar.zst](https://pub-3cca138de6c349f8afe5f6635f9f6f81.r2.dev/data/testnet-chaindata-20240125.tar.zst) - md5: 185b331a21fad339fbdb2f6c27b913a0

### Usage

Step 1: Preparation
- Make sure your hardware meets the [suggested requirement](https://docs.roninchain.com/docs/node-operators/mainnet/non-validator#install-the-node).
- A disk with enough free storage, at least twice the size of the snapshot.

Step 2: Download & Uncompress
- Copy the above snapshot URL.
- Download:  `wget -O chaindata.tar.zst  "<paste snapshot URL here>"` . It will take one or two hours to download the snapshot, you can put it in to the `tmux` by `wget -O chaindata.tar.gz  "<paste snapshot URL here>"`


* [OPTIONAL] If you need to speedup download, just use [aria2c](https://github.com/aria2/aria2)
```
aria2c -o chaindata.tar.zst -s14 -x14 -k100M https://pub-3cca138de6c349f8afe5f6635f9f6f81.r2.dev/data/{filename}
```

But aria2c may fail sometimes, you need to rerun the download command. To make it convient, you can use the following script, save it into file `download.sh`, open new `tmux` session and run: `chmod +x download.sh && ./download.sh "<paste snapshot URL here>" <your dir>`
```
#!/bin/bash
if [ $# -eq 1 ]; then
        dir=$(pwd)
elif [ $# -eq 2 ]; then
        dir=$2
else
        echo "Usage: $0 <uri> [filepath] "
        exit 1
fi
uri=$1
filename=$(basename "$uri")
status=-1
while (( status != 0 ))
do
        PIDS=$(pgrep aria2c)
        if [ -z "$PIDS" ]; then
                aria2c -d $dir -o $filename -s14 -x14 -k100M $uri
        fi
        status=$?
        pid=$(pidof aria2c)
        wait $pid
        echo aria2c exit.
        case $status in
                3)
                        echo file not exist.
                        exit 3
                        ;;
                9)
                        echo No space left on device.
                        exit 9
                        ;;
                *)
                        continue
                        ;;
        esac
done
echo download succeed.
exit 0
```

- Performance pretty good compare to `wget` command:

```
[#daede1 145GiB/145GiB(99%) CN:1 DL:115MiB]
10/05 10:34:40 [NOTICE] Download complete: /axie/geth.tar.zst

Download Results:
gid   |stat|avg speed  |path/URI
======+====+===========+=======================================================
daede1|OK  |   207MiB/s|/axie/geth.tar.zst

Status Legend:
(OK):download completed.

real    12m2.862s
user    1m57.320s
sys     2m28.624s
```

- Uncompress: `tar -I zstd -xvf chaindata.tar.zst`. It will take more than 20 min to uncompress. You can put it in the `tmux` session and run command `tar -I zst -xvf chaindata.tar.zst`
- You can combine the above steps by running a script:

```
wget -O chaindata.tar.zst  "<paste snapshot URL here>"
tar -I zstd -xvf chaindata.tar.zst
```


- If you do not need to store the archive for use with other nodes, you may also extract it while downloading to save time and disk space:
```
wget -q -O - <snapshot URL> | tar -I zstd -xvf -
```


Step 3: Install the node
- Now you can follow steps by steps from here [Install the node ](https://docs.roninchain.com/docs/node-operators/mainnet/non-validator#install-the-node)
- This docs is the detail for `6.(Optional) Download the snapshot`



## Archive Node Data Snapshot

> For more granular upload & download to avoid big files error, split the files into several chunks, so please download them together and concatenate finally.
### Endpoint(Testnet): update every 6 months

- archive-testnet: [archive-testnet-chaindata-20240124.tar.zst](https://pub-3cca138de6c349f8afe5f6635f9f6f81.r2.dev/data/archive-testnet-chaindata-20240124.tar.zst) - md5: 185b331a21fad339fbdb2f6c27b913a0













### Endpoint(Mainnet): update every 3 months
- archive-mainnet-000: [archive-mainnet-chaindata-20240126.tar.zst-000](https://pub-3cca138de6c349f8afe5f6635f9f6f81.r2.dev/data/archive-mainnet-chaindata-20240126.tar.zst-000) - md5: 15e9f506486c42f74e504bcdc056c834

### Endpoint (Mainnet full data) [from: 0 - to: 31.476.532]:
- archive-mainnet: [archive-mainnet-chaindata-20240126.tar.zst](https://pub-3cca138de6c349f8afe5f6635f9f6f81.r2.dev/data/archive-mainnet-chaindata-20240126.tar.zst) - md5: 15e9f506486c42f74e504bcdc056c834

### Usage
Download && Concatenate && Uncompress
```shell
aria2c -s14 -x14 -k100M https://pub-3cca138de6c349f8afe5f6635f9f6f81.r2.dev/data/archive-mainnet-chaindata-20240125.tar.zst-00[0]..archive-mainnet-chaindata-20240125.tar.zst-00[]
cat "archive-mainnet-chaindata-20240125.tar.zst-"* > combined_compressed_file.tar.zst
tar -I zstd -xvf chaindata.tar.zst
```