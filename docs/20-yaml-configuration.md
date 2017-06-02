# blobxfer YAML Configuration
`blobxfer` accepts YAML configuration files to drive the transfer. YAML
configuration files are specified with the `--config` option to any
`blobxfer` command.

## Schema
The `blobxfer` YAML schema consists of 5 distinct "sections". The following
sub-sections will describe each. You may combine all 5 sections into the
same YAML file if desired as `blobxfer` will only read the required sections
to execute the specified command.

#### Configuration Sections
1. [`azure_storage`](#azure-storage)
2. [`options`](#options)
3. [`download`](#download)
4. [`upload`](#upload)
5. [`synccopy`](#synccopy)

### <a name="azure-storage"></a>`azure_storage`
The `azure_storage` section specifies Azure Storage credentials that will
be referenced for any transfer while processing the YAML file. This section
is required.

```yaml
azure_storage:
    endpoint: core.windows.net
    accounts:
        mystorageaccount0: ABCDEF...
        mystorageaccount1: ?se...
```

* `endpoint` specifies for which endpoint to connect to with Azure Storage.
Generally this can be omitted if using Public Azure regions.
* `accounts` is a dictionary of storage account names and either a
storage account key or a shared access signature token.

### <a name="options"></a>`options`
The `options` section specifies general options that may be applied across
all other sections in the YAML configuration.

```yaml
options:
    log_file: /path/to/blobxfer.log
    resume_file: /path/to/resumefile.db
    progress_bar: true
    verbose: true
    timeout_sec: null
    concurrency:
        md5_processes: 2
        crypto_processes: 2
        disk_threads: 16
        transfer_threads: 32
```

* `log_file` is the location of the log file to write to
* `resume_file` is the location of the resume database to create
* `progress_bar` controls display of a progress bar output to the console
* `verbose` controls if verbose logging is enabled
* `timeout_sec` is the timeout to apply to requests/responses
* `concurrency` is a dictionary of concurrency limits
  * `md5_processes` is the number of MD5 offload processes to create for
    MD5 comparison checking
  * `crypto_processes` is the number of decryption offload processes to create
  * `disk_threads` is the number of threads for disk I/O
  * `transfer_threads` is the number of threads for network transfers

### <a name="download"></a>`download`
The `download` section specifies download sources and destination. Note
that `download` refers to a list of objects, thus you may specify as many
of these sub-configuration blocks on the `download` property as you need.
When the `download` command with the YAML config is specified, the list
is iterated and all specified sources are downloaded.

```yaml
download:
    - source:
        - mystorageaccount0: mycontainer
        - mystorageaccount1: someothercontainer/vpath
      destination: /path/to/store/downloads
      include:
        - "*.txt"
        - "*.bxslice-*"
      exclude:
        - "*.bak"
      options:
          check_file_md5: true
          chunk_size_bytes: 16777216
          delete_extraneous_destination: false
          mode: auto
          overwrite: true
          recursive: true
          rename: false
          restore_file_attributes: true
          rsa_private_key: myprivatekey.pem
          rsa_private_key_passphrase: myoptionalpassword
          skip_on:
              filesize_match: false
              lmt_ge: false
              md5_match: true
    - source:
        # next if needed...
```

* `source` is a list of storage account to remote path mappings
* `destination` is the local resource path
* `include` is a list of include patterns
* `exclude` is a list of exclude patterns
* `options` are download-specific options
  * `check_file_md5` will integrity check downloaded files using the stored MD5
  * `chunk_size_bytes` is the maximum amount of data to download per request
  * `delete_extraneous_destination` will cleanup any files locally that are
    not found on the remote. Note that this interacts with include and
    exclude filters.
  * `mode` is the operating mode
  * `overwrite` specifies clobber behavior
  * `recursive` specifies if remote paths should be recursively searched for
    entities to download
  * `rename` will rename a single entity source path to the `destination`
  * `restore_file_attributes` will restore POSIX file mode and ownership if
    stored on the entity metadata
  * `rsa_private_key` is the RSA private key PEM file to use to decrypt
    encrypted blobs or files
  * `rsa_private_key_passphrase` is the RSA private key passphrase, if required
  * `skip_on` are skip on options to use
    * `filesize_match` skip if file size match
    * `lmt_ge` skip if local file has a last modified time greater than or
      equal to the remote file
    * `md5_match` skip if MD5 match

### <a name="upload"></a>`upload`
The `upload` section specifies upload sources and destinations. Note
that `upload` refers to a list of objects, thus you may specify as many
of these sub-configuration blocks on the `upload` property as you need.
When the `upload` command with the YAML config is specified, the list
is iterated and all specified sources are uploaded.

```yaml
upload:
    - source:
        - /path/to/hugefile1
        - /path/to/hugefile2
      destination:
        - mystorageaccount0: mycontainer/vdir
        - mystorageaccount1: someothercontainer/vdir2
      include:
        - "*.bin"
      exclude:
        - "*.tmp"
      options:
          mode: auto
          chunk_size_bytes: 0
          delete_extraneous_destination: true
          one_shot_bytes: 33554432
          overwrite: true
          recursive: true
          rename: false
          rsa_public_key: mypublickey.pem
          skip_on:
              filesize_match: false
              lmt_ge: false
              md5_match: true
          store_file_properties:
              attributes: true
              md5: true
          strip_components: 1
          vectored_io:
              stripe_chunk_size_bytes: 1000000
              distribution_mode: stripe
    - source:
        # next if needed...
```

* `source` is a list of local resource paths
* `destination` is a list of storage account to remote path mappings
* `include` is a list of include patterns
* `exclude` is a list of exclude patterns
* `options` are upload-specific options
  * `mode` is the operating mode
  * `chunk_size_bytes` is the maximum amount of data to upload per request.
    This corresponds to the block size for block and append blobs, page size
    for page blobs, and the file chunk for files. Only block blobs can have
    a block size of up to 100MiB, all others have a maximum of 4MiB.
  * `one_shot_bytes` is the size limit to upload block blobs in a single
    request.
  * `overwrite` specifies clobber behavior
  * `recursive` specifies if local paths should be recursively searched for
    files to upload
  * `rename` will rename a single entity destination path to a single `source`
  * `rsa_public_key` is the RSA public key PEM file to use to encrypt files
  * `skip_on` are skip on options to use
    * `filesize_match` skip if file size match
    * `lmt_ge` skip if remote file has a last modified time greater than or
      equal to the local file
    * `md5_match` skip if MD5 match
  * `store_file_properties` stores the following file properties if enabled
    * `attributes` will store POSIX file mode and ownership
    * `md5` will store the MD5 of the file
  * `strip_components` is the number of leading path components to strip
  * `vectored_io` are the Vectored IO options to apply to the upload
    * `stripe_chunk_size_bytes` is the stripe width for each chunk if `stripe`
      `distribution_mode` is selected
    * `distribution_mode` is the Vectored IO mode to use which can be one of
      * `disabled` will disable Vectored IO
      * `replica` which will replicate source files to target destinations on
        upload. Note that more than one destination should be specified.
      * `stripe` which will stripe source files to target destinations on
        upload. If more than one destination is specified, striping occurs in
        round-robin order amongst the destinations listed.

### <a name="synccopy"></a>`synccopy`
TODO: not yet implemented.