# Parallel Directory Copier

A Jupyter Notebook utility for copying multiple large directories from one storage location to another with progress reporting, retry logic, and basic quality checks.

This project is useful when you need to copy large datasets across network/shared storage, for example from one SMB/network share to another.

## Main features

- Copies multiple source directories into one destination directory.
- Uses parallel workers, tuned automatically from system resources, network speed, and average file size.
- Handles very large files with a fixed 64 MiB buffer.
- Limits simultaneous open file handles to reduce `errno 24: Too many open files` errors.
- Retries failed file copies with exponential backoff.
- Uses a heartbeat check to detect temporary network loss and resume when the destination becomes reachable again.
- Skips files that were already copied when the destination file has the same size.
- Optional quality check modes:
  - `fast`: compares file count and total size.
  - `hash`: compares SHA-256 hashes for stronger verification, but much slower.
- Shows notebook progress bars when `tqdm` is installed.
- Saves a CSV report at the destination after the run.

## Files in this repository

```text
Directories_copier_script_v5.ipynb   # Main Jupyter Notebook
README.md                            # Project documentation
```

## Requirements

Install Python 3.9 or newer. The notebook uses the following Python packages:

```bash
pip install pandas psutil tqdm jupyter
```

`tqdm` is optional. If it is not installed, the notebook still runs but progress bars are disabled.

## How to use

1. Open `Directories_copier_script_v5.ipynb` in Jupyter Notebook, JupyterLab, VS Code, or another notebook editor.

2. Edit the **User settings** cell:

```python
SOURCE_DIRECTORIES = [
    r"\\from-storage\dir1",
    r"\\from-storage\dir2",
    r"\\from-storage\dir3"
]

DESTINATION_DIRECTORY = r"\\to_storage\dir1"
```

Replace the example paths with your real source and destination paths.

3. Choose a quality check mode:

```python
QUALITY_CHECK_MODE = "fast"
```

Use `fast` for most large copies. Use `hash` only when you need the strongest verification and can accept a much slower check.

4. Run the notebook cells from top to bottom.

5. After the copy finishes, check the summary table and the CSV report saved in the destination directory.

## Important safety notes

Before running the notebook on important data:

- Test it first with a small folder.
- Confirm that the destination path is correct.
- Make sure the destination has enough free space.
- Keep `SKIP_ALREADY_COPIED = True` if you want resumable behavior.
- Avoid running multiple copies to the same destination at the same time.

## Configuration options

| Option | Meaning |
|---|---|
| `SOURCE_DIRECTORIES` | List of folders to copy. |
| `DESTINATION_DIRECTORY` | Parent folder where copied folders will be created. |
| `QUALITY_CHECK_MODE` | `fast` or `hash`. |
| `SKIP_ALREADY_COPIED` | Skip folders/files that appear already copied. |
| `PRESERVE_METADATA` | Preserve file modification/access times when possible. |
| `MAX_RETRIES` | Number of retry attempts per file. |
| `RETRY_BASE_DELAY` | Initial retry delay in seconds. |
| `HEARTBEAT_INTERVAL_SEC` | How often the destination is checked. |
| `COPY_BUFFER_SIZE` | File copy buffer size. Default is 64 MiB. |
| `MAX_OPEN_FILE_PAIRS` | Maximum simultaneous source/destination file pairs. |
| `MAX_WORKERS_OVERRIDE` | Manually set worker count, or leave as `None` for automatic tuning. |

## Output

The notebook prints progress while copying and creates a final summary table with:

- status
- source path
- destination path
- files copied
- files skipped
- copied size in GB
- throughput in MB/s
- elapsed time
- message

It also saves a CSV report similar to:

```text
copy_report_v4.csv
```

inside the destination directory.

## Troubleshooting

### `No valid source directories found`

Check that all paths in `SOURCE_DIRECTORIES` exist and are accessible from the computer running the notebook.

### `Not enough free disk space at destination`

Free space on the destination drive or choose a different destination.

### Network copy stops or fails

The notebook includes heartbeat and retry logic, but unstable network shares can still fail. Re-run the notebook after the connection is restored. Already copied files should be skipped when `SKIP_ALREADY_COPIED = True`.

### Copy is slow

Large files are often limited by disk or network speed. The notebook automatically reduces worker count for very large average file sizes to avoid overloading the disk.

## License

Add a license before sharing publicly. For open-source projects, common choices include MIT, Apache-2.0, and GPL-3.0.

## Author

Created by Dario Ricca.
