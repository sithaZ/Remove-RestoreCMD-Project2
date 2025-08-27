# Trash-Enabled rm and Restore System

This project replaces the standard, permanent `rm` command with a safer alternative that moves files to a temporary trash directory. It includes a robust logging system and an interactive command to easily restore deleted files, preventing accidental data loss.



## 🌟 Features

* **Safe Deletion**: Instead of permanent deletion, files and directories are moved to `/tmp/trash/`.
* **Recursive Deletion**: Supports the `-r` flag to safely delete directories.
* **Structured Logging**: All deletions are recorded in `/tmp/trash/trash.log` using a clean, single-line JSON format.
* **Interactive Restore**: An interactive, fuzzy-search menu powered by `fzf` allows for quick and easy file recovery.
* **Environment Override**: Seamlessly integrates into the shell to override the default `rm` command.

## 🔧 Prerequisites

Before you begin, ensure you have the following command-line tools installed.

* **fzf**: For the interactive restore menu.
* **jq**: For creating and parsing the JSON-formatted logs.

You can install them on a Debian-based system (like Ubuntu) with:
```bash
sudo apt update && sudo apt install fzf jq
```

## ⚙️ Installation

1.  **Place Scripts**: Copy the `rm` and `restore` scripts into a personal binary directory.
    ```bash
    mkdir -p ~/bin
    # ...copy rm and restore scripts into ~/bin/...
    ```

2.  **Make Executable**: Grant execute permissions to both scripts.
    ```bash
    chmod +x ~/bin/rm ~/bin/restore
    ```

3.  **Update PATH**: Add your personal `bin` directory to your shell's `PATH` to ensure your scripts are used instead of the system defaults. This command adds the line to your `.bashrc` file.
    ```bash
    echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
    ```

4.  **Reload Shell**: To apply the changes, either open a new terminal or run:
    ```bash
    source ~/.bashrc
    ```

## 🚀 Usage

#### Deleting Files

The `rm` command works just like the original, but much more safely.

* **Delete a file**:
    ```bash
    rm my_file.txt
    ```

* **Delete a directory**:
    ```bash
    rm -r my_project_folder/
    ```

#### Restoring Files

To restore a file, simply run the `restore` command without any arguments.

```bash
restore
```
This will open an interactive menu. Use the **arrow keys** to navigate, **type to filter** the list, and press **Enter** to restore the selected file.

## 🛠️ How It Works

* **`rm` Script**: The script parses arguments, rejecting any unsupported flags. For each valid file or directory, it generates a unique name by appending a Unix timestamp and moves it to `/tmp/trash/`. It then uses `jq -c` to generate a compact, single-line JSON object and appends it to `/tmp/trash/trash.log`.

* **`restore` Script**: This script reads the `trash.log` file and pipes the contents to `fzf`, creating the interactive menu. When a user selects a line, the script pipes that JSON string to `jq` to extract the `originalPath` and `trashPath`. It then checks for conflicts and moves the file back to its original location, recreating parent directories if necessary.

## 📂 File Structure

The system relies on a simple directory and file structure located in `/tmp/` and the user's home directory.

```
/tmp/trash/
├── trash.log                 # The JSON log of all deletions
└── [deleted_files_with_timestamps]

~/bin/
├── rm                        # The custom safe rm script
└── restore                   # The interactive restore script
```

## ✨ Optional Enhancements

#### Auto-Cleanup

To prevent the trash directory from growing indefinitely, you can set up a `cron` job to automatically delete files older than a certain number of days (e.g., 30 days).

1.  Open the crontab editor:
    ```bash
    crontab -e
    ```

2.  Add the following line to run a cleanup command every day at 2:00 AM:
    ```crontab
    0 2 * * * find /tmp/trash/ -type f -mtime +30 -delete
    ```
