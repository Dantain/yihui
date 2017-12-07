---
title: Frequently (?) Asked Questions
date: '2017-12-02'
---

TinyTeX is still a relatively new project, so these are only potential FAQs.

1. **Which version of TeX Live is installed?**

    The (very) latest. Using the very latest version of a software package can be risky. You have been warned. On the other hand, however, let's not pretend we haven't run into problems using the "stable" versions of TeX Live.
    
    Latest or stable? Your call. If you are not comfortable with the latest version, you should not consider TinyTeX.

1. **Do you provide prebuilt binaries of TinyTeX?**

    No. Technically it is easy, but I don't really understand the implications of [the TeX Live license](https://www.tug.org/texlive/LICENSE.TL). Specifically, the license says:
    
    > [...] TeX Live has neither a single copyright holder nor a single license covering its entire contents, since it is a collection of many independent packages.  Therefore, you may copy, modify, and/or redistribute software from TeX Live only if you comply with the requirements placed thereon by the owners of the respective packages.

    That sounds complicated to me.

1. **What is the size of TinyTeX?**

    About 150MB on macOS and Ubuntu, and 220MB on Windows (when installed). You may think it is still too big, but please consider that the size of [BasicTeX](https://www.tug.org/mactex/morepackages.html) for macOS is about 215MB (when installed), and a [basic MiKTeX installer](https://miktex.org/download) for Windows is about 750MB (I didn't check how big it is when installed).
    
    If you create a tarball of TinyTeX on macOS or Ubuntu, it will be only 50MB. This can be very helpful if you install such a tarball on the cloud (e.g., for software testing purposes on Travis CI). The download and installation should take only a few seconds.

1. **What does the TinyTeX installation script do exactly? How do you reduce the size of the gigantic TeX Live?**

    The best way to understand TinyTeX is to [read the source](https://github.com/yihui/tinytex/) under the `tools` directory. Basically, TinyTeX automates the TeX Live installation using a profile file named [texlive.profile](https://github.com/yihui/tinytex/blob/master/tools/texlive.profile) (`./install-tl -profile=texlive.profile`), which only specifies the `infraonly` scheme to be installed first. With this scheme, you cannot really compile any LaTeX documents, but it contains the most important utility `tlmgr` (TeX Live Manager). At this point, the total size is about 80MB.
    
    Then I use `tlmgr` to install a few more commonly used packages (defined in [pkgs-custom.txt](https://github.com/yihui/tinytex/blob/master/tools/pkgs-custom.txt)). With these packages, you should be able to compile most R Markdown documents to PDF. The total size becomes about 150MB.
    
    The fact that I only included a small number of LaTeX packages in TinyTeX is one of the two reasons why TinyTeX is relatively small in size. The other reason is that I excluded the source files and documentation of packages. In `texlive.profile`, you can see these two options:

    ```config
    option_doc 0
    option_src 0
    ```

    Why do I exclude source files? Because they are unlikely to be useful to end-users. Would you really read the source code of a LaTeX package? Probably not, unless you are a developer or advanced LaTeX user. In fact, this also explains why I don't provide prebuilt binaries of TinyTeX: some open-source licenses may require that you provide source files when you redistribute the open-source software. TinyTeX does not redistribute TeX Live, but only provides a custom installation script.

    Why do I exclude the documentations? Tell me honestly: how many times have you found a solution via [StackExchange](https://tex.stackexchange.com), and how many times have you tried to read the package documentation? Even with the full documentation installed, you probably don't even know where to find these documentation files on your computer. The documentation files take a lot of disk space, and I believe they are rarely read by an average user, so they are not included. The address bar of your web browser is the most convenient documentation: type and search.

    The other major factor that affects the size of TeX Live is the font packages, which are usually much bigger than other LaTeX packages, but we cannot really do much about it, unless you do not use `pdflatex`, in which case you may further reduce the size of this small TeX Live distribution.

1. **I'm an R package developer. Are the default LaTeX packages included in TinyTeX enough for me to develop an R package?**

    No, you need a few more: **metafont**, **mfware**, **inconsolata**, and **tex**, mainly because `R CMD check` needs to build manuals to PDF. You can install these packages through either the command line:
    
    ```sh
    tlmgr install metafont mfware inconsolata tex
    tlmgr path add
    ```
    
    or via the R function:
    
    ```r
    tinytex::tlmgr_install(c('metafont', 'mfware', 'inconsolata', 'tex'))
    ```
    
    Another important thing to do is add R's texmf tree to TeX Live. This is not easy to explain, but I have provided a function `tinytex::r_texmf()` to do it. Just do it. See the help page `?tinytex::r_texmf` if you are curious about the technical details.
    
    Although it is irrelevant to TinyTeX, you may also need to install [**texinfo**](https://www.gnu.org/software/texinfo/) (not a LaTeX package), since `R CMD check` may also require it. For macOS users, you can install it via Homebrew: `brew install texinfo`.

1. **I'm a Linux system admin. How can I install TinyTeX for all users of a system?**

    First, add two options `--admin --no-path` to the installation script:

    ```sh
    wget -qO- \
      "https://github.com/yihui/tinytex/raw/master/tools/install-unx.sh" | \
      sh -s - --admin --no-path
    ```
    
    This will install TinyTeX to `~/.TinyTeX`, and this step does _not_ require root privileges. Then you need to add symlinks to `/usr/local/bin` via `sudo`, so that all users of this system can use the TeX Live executables (e.g., `pdflatex`):
    
    ```sh
    sudo ~/.TinyTeX/bin/*/tlmgr path add
    ```

    If you want to move `~/.TinyTeX` to a different location, see FAQ 9, and remember to run `tlmgr path add` with `sudo` after you move the folder, to move sure symlinks under `/usr/local/bin` point to the the new paths correctly.

1. **I'm a Linux admin. I used the above approach to install TinyTeX and added symlinks to `/usr/local/bin`. How can my users without root privileges install LaTeX packages by themselves?**

    Users without root privileges won't be able to install packages via a normal `tlmgr install` command, unless you have assigned them to a group that has write permissions to the TinyTeX root directory (by default, `~/.TinyTeX`, and note that `~` is _your own_ home directory), in which case these users are able to modify the system-wide installation of TinyTeX.
    
    For those who do not have write permissions to the TinyTeX directory, there is a [user mode](https://www.tug.org/texlive/doc/tlmgr.html#USER-MODE) in TeX Live that allows such users to manage a user-level texmf tree, e.g., a user can install packages to his/her home directory instead of a system directory. With TinyTeX, the user-level texmf tree is at `~/.TinyTeX/texmf-home` (in TeX Live's terms, this is the `TEXMFHOME` variable, and note that `~` is the _user's_ home directory).
    
    The first thing users have to do is to initialize the tree (create the `TEXMFHOME` directory if it does not exist). It only needs to be done once:
    
    ```sh
    tlmgr init-usertree
    ```
    
    Then when they install packages, they must always use the option `--usermode`, e.g.,
    
    ```sh
    tlmgr --usermode install koma-script xcolor
    ```
    
    For R users, the above commands are equivalent to:
    
    ```r
    tinytex::tlmgr('init-usertree')  # again, only do this once
    tinytex::tlmgr_install(c('koma-script', 'xcolor'), usermode = TRUE)
    ```
    
    However, the user mode of TeX Live can actually be quite complicated, and unfortunately it is not something that I can help with. A few known caveats:
    
    - The worst thing is that users cannot install all packages. For TeX Live, some packages are _relocatable_, and some are not. For example, packages containing executables are not relocatable (e.g., the **metafont** package contains the executable `mf`). If users have to use these packages, only the system admin can help. The good news is that the number of such packages is relatively small, so a conservative strategy is to just pre-install all of them.
    
    - Some packages may require running [`updmap`](https://www.tug.org/texlive/doc/updmap.html) after installation (e.g., font packages for `pdflatex`). Good news is that users should be able to run `updmap-user`, and bad news is that whenever the system admin runs `updmap-sys`, users may have to run `updmap-user` again (if I understand the documentation correctly). For users, the conservative stragety is to run `updmap-user` again when they run into font problems that didn't exist previously (R users can run `system2('updmap-user')`).

1. **I'm a Debian/Ubuntu user. How do I prevent TeX Live from being installed when installing other packages that depend on TeX Live? I don't want (or need) both TinyTeX and the official TeX Live packages to be installed at the same time.**

    Yes, `apt-get install` might install certain `texlive` packages. You can download and install [texlive-local.deb](https://travis-bin.yihui.name/texlive-local.deb) to "fool" `apt-get`.
    
    ```sh
    wget "https://travis-bin.yihui.name/texlive-local.deb"
    sudo dpkg -i texlive-local.deb
    rm texlive-local.deb
    ```
    
    This package was built by running [`equivs-build`](http://manpages.ubuntu.com/manpages/trusty/man1/equivs-build.1.html) on [debian-control-texlive-in.txt](https://github.com/scottkosty/install-tl-ubuntu/blob/master/debian-control-texlive-in.txt) (credits to Scott Kostyshak). Basically it pretends all `texlive` packages have been installed (which is not true), so that you can have the full freedom (and responsibility!) of choosing which LaTeX packages to install by yourself.

    Personally, I don't like the way that all LaTeX packages are distributed on Linux (e.g., Debian) as `texlive-*` packages, and each `texlive-*` package contains several LaTeX packages. That means you must install several other LaTeX packages even if you only need one. Imagine as an R user (or Python, or other languages): if someone prebuilds the more than 10,000 R packages on CRAN as 20 _collections_ of packages, what do you feel when you only need the **ggplot2** package but are forced to install all possible packages related to **ggplot2**? Obviously the advantage of providing these collections is that you may not need to install missing packages over and over again (who knows), but I'd prefer a lean TeX Live distribution and install packages by myself (which is not difficult at all to me).

1. **Can I change the installation directory?**

    The directory path is hard-coded in the installation script, and you cannot change it from the command line when installing TinyTeX. I chose these directories for Linux, macOS, and Windows because they are hidden by default on these platforms. TeX Live should not need (cry for) your attention in most cases. If you really want to change the directory, there are two ways:
    
    - You can download the installation scripts and modify them by yourself (in the spirit of open source).
    
    - Or install TinyTeX first, and move the installation folder to where you'd like it to be (you can even move it to a USB stick). That is because TinyTeX is essentially a _portable_ version of TeX Live. The tricky part of this way is to deal with the `PATH` variable. You need to run the command `tlmgr path add`, but `tlmgr` won't be on `PATH` if you have moved the default installation directory, so you have to execute `tlmgr` through its full path, e.g.,
    
        ```sh
        # assume you moved ~/.TinyTeX on Linux to /opt/tinytex
        /opt/tinytex/bin/*/tlmgr path add
        
        # assume you moved ~/Library/TinyTeX on macOS to /opt/tinytex
        /opt/tinytex/bin/*/tlmgr path add
        
        # assume you moved %APPDATA%\TinyTeX on Windows to C:\Software\TinyTeX
        "C:\Software\TinyTeX\bin\win32\tlmgr" path add
        ```
        
        You only need to do this once, and if you installed TinyTeX using the approach in FAQ 6, you need `sudo` to run `tlmgr path add`.

1. **Why doesn't the default installation path contain the year of TeX Live?**

    TeX Live installs itself to a directory named by the year by default (so does MacTeX), e.g., `/usr/local/texlive/2017`. I don't think it makes much sense to average users. Who would want multiple versions of TeX Live to be installed on the same computer except TeX Live developers? The full TeX Live is gigantic, and you probably do not want it to eat your disk space year by year.
    
    Again, if you want to change the installation path of TinyTeX, you certainly can.

1. **Can I use Homebrew to install TinyTeX on macOS?**

    Yes, but there are no obvious advantages, so I don't really recommend this approach:

    ```sh
    brew install -v yihui/tinytex/tinytex
    ```

    The formula is in the repo [yihui/homebrew-tinytex](https://github.com/yihui/homebrew-tinytex) on Github. If you must use this approach, here are two caveats that you should know:
    
    - Homebrew manages the symlinks of TeX Live (via `brew link tinytex`), and they are created under `/usr/local/bin`. You'd better not run `tlmgr path add` afterwards, because this command creates symlinks to the same location. When you install a TeX Live package (via `tlmgr install`) that contains binaries, such as **metafont** (which contains the executable `mf`), you should run:
    
        ```sh
        brew postinstall tinytex && brew unlink tinytex && brew link tinytex
        ```

    - After you update **tinytex** in Homebrew (e.g., via `brew update && brew upgrade`), the LaTeX packages that you previously installed by yourself will no longer be available, and you have to install them again. One solution is to `brew pin tinytex`, so that this formula won't be upgraded in the future (in fact, there should be no need to upgrade it). The other solution is to use the HEAD version of the formula:
    
        ```sh
        brew cleanup -s
        brew remove tinytex
        brew install -v --HEAD tinytex
        ```

    If you feel these issues are too complicated (yes they are), just follow the simpler way on the [homepage](/tinytex/) to install TinyTeX on macOS.

1. **How to open a command window (terminal) to execute the commands you mentioned?**

    I'm not going to answer this question for Linux users. For macOS users, `Command + Space` to launch the Spotlight Search, and type `Terminal`. The first result should be `Terminal.app`. That is it. For Windows users, click the `Start` menu, select `Run`, and type `cmd`.
    
    If you are an [RStudio](https://www.rstudio.com) user, it can be easier. RStudio (>= v1.1) has [built-in support for terminals](https://blog.rstudio.com/2017/08/11/rstudio-v1-1-preview-terminal/), and you can open a terminal right inside RStudio.