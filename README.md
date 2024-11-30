# Git and Google Drive interop without LFS

This document introduces a way to use Git and Google Drive together without using Git LFS. 

This is useful when you have a large file that you want to version control with Git, but you don't want to use Git LFS. (Especially having quite large files, such as game assets or movie files.)

NOTE: This method is not dependent on Google Drive. You can use any other (cloud/server/local) storage instead of Google Drive. And also you can use this method "with" Git LFS. (ex: when you want to use Git LFS for most files, but some quite large files are not suitable for Git LFS.)

## Limitations

- In this method, copying and transferring files is using other methods. So you can detect the modification of the files, but which file should be copied or transferred is not automatically done.
  - So this method **is not suitable for the case** that you want to frequently back to the previous version of the large file. (In this case, you can also consider [git-lfs-agent-rclone](https://github.com/funatsufumiya/git-lfs-agent-rclone), [git-lfs-agent-scp](https://github.com/funatsufumiya/git-lfs-agent-scp), [git-lfs-php-server](https://github.com/funatsufumiya/git-lfs-php-server) or other methods.)
- The advantage gained in exchange for this limitation is, that **file names can be handled as normal**. This is desirable, in my opinion, for management in Google Drive and Dropbox.
  - This method is especially useful in situations where there are old-fashioned _v2 or _final file names in Google Drive or Dropbox, and files with the same name are occasionally updated. ( I put aside whether old fashioned file names are really 
acceptable or go nice with git or not... )

## Script(s)

- [scripts/large-file-checker](scripts/large-file-checker)
- [scripts/large-file-checker.nu](scripts/large-file-checker.nu) (Nushell version)

## How it works (concept)

1. Copy the large file from Google Drive to your local machine.
2. Just ignore the large file in your `.gitignore` file.
3. You can track modification of the ignored file by using the `large-file-checker` script.
4. You should track `list.txt` and `hash.txt` in the directory where the large file is located. (which is generated by the `large-file-checker` script)
5. In another environment (or another time), you can check the modification or list the files that are not in sync with Google Drive.
6. Copy files again from Google Drive to your local machine if necessary, by hands or `rsync` from mounted Google Drive (using `rclone` or public Google Drive apps for PC).

This process solves the most common problem of using Git and Google Drive together without using Git LFS. Especially for game development or some other large file projects.

## Script detail

The default `large-file-checker` script is adjusted for [openFrameworks](https://openframeworks.cc/) case (ex: `bin/data/**/from_gdrive` is considered as a large file directory), but you can easily modify it for your own case, not only for openFrameworks. (I'm also using [old version of this script](appendix/godot_scripts) for my [Godot Engine](https://godotengine.org/) game project. This was start point of this script.)

In current adjusted version:

- `large-file-checker update assets` will update `list.txt` and `hash.txt` in the `bin/data/assets/from_gdrive` directory, by checking current files in the directory.
- `large-file-checker check assets` will check the modification of the files in the `bin/data/assets/from_gdrive` directory, by comparing `list.txt` and `hash.txt` with the current files in the directory.

This script is created to be able to use from several people in the same project and repository, so `ids` parameter on `large-file-checker` can be used to distinguish the folders in Google Drive and local. (each folder can contain also folders in it.)

So you should modify `ids`, `dir_pattern_for_help` parameter and `get_large_file_dir` function in the script for your own case.

## Notes

- You need `rgh` command to use the script. You can install it by `cargo install rustgenhash`.
- `large-file-checker` just checks local files only. Not communicates with Google Drive. So you can use any method like FTP or Dropbox instead of Google Drive.
- Why `list.txt` and `hash.txt` separated is, this method sometimes users (mostly I) would forget updating hash. In this case, you can also use `large-file-checker check list assets` to check only the list. So you can modify `list.txt` also by hands without hash checking. (This is also a remnant of the coexistence of .import files and list.txt in [the older Godot version script](appendix/godot_scripts).)

## License

WTFPL

(Please create your original version, without notifying me. Feel free to modify and copy this document and scripts. And if you need a regular license, such as the Apache license, you can fork it and do so.)