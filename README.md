# PowerCMD

Batch is limited and lacks many of the programming paradigms found in modern languages, but it's user-friendly for the average user to run. There are also some straightforward advantages to using Batch over PowerShell.

PowerShell, on the other hand, is a more complex scripting language designed with the paradigms of modern programming languages. However, it's not immediately obvious how to run scripts with PowerShell for the broad demographic of Windows users. Therefore, a bundler was created to seamlessly integrate PowerShell within Batch.

Essentially, this approach involves using a minimal amount of Batch code to execute your PowerShell script, and it even allows for the granting of admin privileges.

PowerCMD is a fork of the bundler developed for my [qbactivator](https://github.com/neuralpain/qbactivator) project, prepared as a template for developers to use in their own PowerShell scripting projects.

## How it bundles

Bundling is pretty simple:

  1. Write the batch code necessary to run the PowerShell code (with admin if required) into the `cmd_cache.cmd`
  2. Collect all the PowerShell functions and code and write it into the `pwsh_cache.ps1`
  3. Write the cache files to `script.cmd` in `/dist`

Essentially, there are two separate cache files which are then combined into a single script.

For release packages, you will use the `-s` switch and `-s -a` to compress the release. If compression is enabled, then both a `complete_release` and a `lightweight_release` will be created. The `zip` package is required for compression.

> [!NOTE]  
> - `complete_release` should contain all the files that the script needs to run and function correctly, not integrated with the script itself by ASCII methods i.e. [Compressed2Text](https://github.com/AveYo/Compressed2TXT).
> - `lightweight_release` should only contain files that the user is unable to download or access on their own.

If you are testing iterations of your script, you can use the `-t` switch to create a test in the `/build` folder. Optionally, you are able to add a note within the file name by typing a description like this: `-t testing bug fix`. Each test is created with a unique build number.

> [!TIP]  
> For admin privileges, use the `--with-admin` flag after the `-t`, `-s`, or `-s -a` switches. If you want to add a note for your test which has admin privileges enabled, the note must be last: `-t --with-admin testing bug fix`.

## Configuration

This bundler bundler as a semi-configurable script, as it requires input such as the script name, file locations and whatnot. You should only need to change the below lines of code for the most basic of scripts, but feel free to modify it (and the basic folder structure) to suit your needs.

### Basic script information

```Shell
# edit script version in ./VERSION
version=$(<VERSION)
# change name of script
name=script 
# terminal window title
script_title="Script Title"
```

> [!IMPORTANT]  
> The `VERSION` should be updated on every release of your script.

### Additional files that the script needs to access

```Shell
# add additional files here
additional_files=(
  "file_1.txt"
  "file_2.txt"
  "file_3.txt"
)
```

> [!IMPORTANT]  
> Ensure to correctly define the location of the folder containing the additional files here...
> 
> ```Shell
> ...
> # location of directories
> src=./src
> res=$src/res
>     ^^^^^^^^
> functions=$src/functions
> ...
> ```
>
> ...since it will affect your results here
> 
> ```Shell
> compress() {
> ...
>   for file in "${additional_files[@]}"; do 
>     cp $res/$file dist
>        ^^^^^^^^^^
> ...
> ``` 

### List PowerShell functions

```Shell
# declare a list of your PowerShell functions here
powershell_functions=(
  "$functions/Function-One.ps1"
  "$functions/Function-Two.ps1"
  "$functions/Function-Three.ps1"
  # you should not need to remove main unless
  # the main PowerShell file is renamed
  "$src/Main.ps1"
)
```

### Edit script head comment

```Shell
bundle() {
  ...
  # add the copyright information, link to your project repository and
  # description of the script, or remove it entirely, whichever you choose
  echo ":: Copyright (c) 1937, Alan Turing" >> $cmd_cache
  echo ":: https://git.kernel.org/pub/scm/" >> $cmd_cache
  echo ":: A script that does stuff" >> $cmd_cache
  ...
}
```

### Add Batch code to be run after PowerShell execution is complete

```Shell
bundle() {
  ...
  # -- add batch code | this is optional -- #
  # cat $src/main.cmd >> $cmd_cache  # optional
  # echo >> $cmd_cache               # optional
  # -- end batch code -- #
  ...
}
```

### Specify files to exclude from (or include in with `-i`) the `lightweight_release`

```Shell
# files to exclude in lightweight release
# `*.zip` is mandatory, else it will include the normal release as well
zip -q $lightweight_release * -x file_2.txt file_3.txt *.zip || [...]
```

> [!TIP]  
> ```
> Include and Exclude:
> -i pattern pattern ...   include files that match a pattern
> -x pattern pattern ...   exclude files that match a pattern
> Patterns are paths with optional wildcards and match paths as stored in
> archive.  Exclude and include lists end at next option, @, or end of line.
>   zip -x pattern pattern @ zipfile path path ...
> ```

## Usage

```
Usage: PowerCMD [OPTION...]
A bundler to integrate PowerShell with CMD\n
-s, --release      Build for stable release
    --with-admin   Include admin permission request
-a, --archive      Archive stable release package
-t, --test [note]  Build unit tests
-C, --clear-all    Delete temporary files and folders
-c, --clear        Clear all unit test builds
-v, --version      Display version number and exit
-h, --help         Display this help message and exit
```

```
--- Neither PowerShell nor Batch was harmed during the development of this project ❤️
```
