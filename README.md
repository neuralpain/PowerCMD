# PowerCMD

Batch lacks many of the programming paradigms found in modern languages, but it's user-friendly for the wider demographic of users to execute. There are also some straightforward advantages to using Batch over PowerShell, which is, on the other hand, a more complex scripting language designed with the paradigms of modern programming languages.

That being said, it's not immediately obvious to the broad demographic of Windows users how you are supposed to run PowerShell scripts (it sure wasn't for me, the first time I wanted to run a PowerShell script) and having to open up a terminal (and possibly jump through some safety hoops) just to run it as opposed to the simple *double-click* of a Batch script is a bit tedious. Therefore, I created a bundler to quickly integrate PowerShell within Batch to take advantage of some of its benefits.

Essentially, this approach involves using a minimal amount of Batch code to execute your PowerShell script, and additionally, it allows for the granting of admin privileges.

> [!NOTE]  
> PowerCMD is a fork of the bundler developed for my [qbactivator](https://github.com/neuralpain/qbactivator) project, prepared as a template which other developers can to use for their own PowerShell scripting projects. The [current implementation](https://github.com/neuralpain/qbactivator/blob/v0.21.1/compile) of the bundler in the *qbactivator* project is outdated and will be replaced with this script in a subsequent release.
> - uses neuralpain / [cmdUAC.cmd](https://gist.github.com/neuralpain/4bcc08065fe79e4597eb65ed707be90d)
> - uses neuralpain / [PwshBatch.cmd](https://gist.github.com/neuralpain/4ca8a6c9aca4f0a1af2440f474e92d05)

## Why should you use PowerCMD?

- **Improves efficiency** by allowing you to work with multiple PowerShell files
- **Improved readability** by splitting up your large script into multiple files to produce smaller, more focused files
- Optionally enable **archiving** of your script and other related files into a ZIP file for release
- Optionally include code for **admin permissions** request
- It's a script that gobbles up code and spits out another script just the way you need it... so why not?

## How it bundles

Bundling is pretty simple:

  1. Writes the batch code necessary to run the PowerShell code (with admin if required) into the `cmd_cache.cmd`
  2. Collects all the PowerShell functions and code and writes it into the `pwsh_cache.ps1`
  3. Writes the cache files to a single `script.cmd` in `/dist`

Essentially, there are two separate cache files which are then combined into a single script.

For release packages, you will use the `-s` switch or `-s -a` to archive the release. If archiving is enabled, then both a `complete_release` and a `lightweight_release` will be created. **The `zip` package is required for archiving.**

> [!NOTE]  
> - `complete_release` should contain all the files that the script needs to run and function correctly, not integrated with the script itself by ASCII methods i.e. [Compressed2Text](https://github.com/AveYo/Compressed2TXT)
> - `lightweight_release` should only contain files that the user is unable to download or access on their own

If you are testing iterations of your script, you can use the `-t` switch to create a test in the `/build` folder. Optionally, you are able to add a note within the file name by typing a description like this: `-t testing bug fix`. Each test is created with a unique build number.

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

### Edit script head comment

```Shell
# license information
license_year="1937"
license_owner="Alan Turing"
# link to software website or oss repo
project_url="https://git.kernel.org/pub/scm/"
# basic description of what this script does
script_description="A script that does stuff"
```

> [!IMPORTANT]  
> The `VERSION` file should be updated on every release of your script.

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

### Add Batch code to be run after PowerShell execution is complete

```Shell
bundle() {
  ...
  # -- add batch code | this is optional -- #
  cat $src/main.cmd >> $cmd_cache
  echo >> $cmd_cache
  # -- end batch code -- #
  ...
}
```

### Specify files to exclude from (or include in with `-i`) the `lightweight_release`

```Shell
# files to exclude in *.min.zip
exclude_files=(
  "file_1.txt"
  "file_3.txt"
)

[...]

zip -q $lightweight_release * -x ${exclude_files[@]} *.zip || [...]
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
A bundler to integrate PowerShell with CMD
-s, --release      Build for stable release
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
