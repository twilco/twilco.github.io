---
layout: post
title:  "RISC-V from scratch 1: Introduction, toolchain setup, and hello world!"
date:   2019-03-10 12:42:53
categories: riscv toolchain
---

{: .no_toc}
#### Table of contents
1. TOC
{:toc}

### Introduction

Welcome to part one of *RISC-V from scratch!*  Throughout this series, we will utilize RISC-V and its ecosystem to explore various low-level tools and concepts, such as linkers, runtimes and startup routines, assembly, and lots more.  I am a web developer by trade, and as such I'm not exposed to these things on a daily basis.  However, I think they are very interesting - hence this series!  Join me on a very much unstructured journey into the depths of all things low-level.

In this first post, we'll talk a little bit about what RISC-V is, why it's important, and finish up by setting up a RISC-V toolchain and building and running a simple RISC-V program.

### So what is RISC-V?

RISC-V is an open-source, free-to-use ISA that began as a project at UC-Berkeley in 2010.  The free-to-use aspect has been instrumental in its success and is quite a stark contrast to many other architectures.  Take ARM for example - in order to create an ARM-compatible processor, you must pay an upfront fee of [$1M - $10M as well as a 0.5% - 2% royalty fee per-chip](https://www.anandtech.com/show/7112/the-arm-diaries-part-1-how-arms-business-model-works/2).  This free and open model makes RISC-V an attractive option to many groups of people - hardware startups who can't foot the bill to create an ARM or other licensing-required processor, academic institutions, and (obviously) the open-source community.

RISC-V's meteoric rise in popularity hasn't gone unnoticed.  [ARM launched a now-taken down website](https://abopen.com/news/rattled-arm-launches-anti-risc-v-marketing-campaign/) that attempted (rather unsuccessfully) to highlight supposed benefits of ARM over RISC-V.  RISC-V is backed by [a ton of major companies](https://riscv.org/members-at-a-glance/), including Google, Nvidia, and Western Digital.

### QEMU and RISC-V toolchain setup

We won't be able to run any code on a RISC-V processor until we have an environment to do it in.  Fortunately, we don't need a physical RISC-V processor to do this - we'll instead be using [qemu-riscv](https://github.com/riscv/riscv-qemu/wiki).  Follow the install instructions for your operating system [here](https://github.com/riscv/riscv-qemu/wiki#build-and-install).  I'm using MacOS, so for me this was as easy as:

{% highlight bash %}
# create a directory to put our risc-v emulators, toolchain, etc
mkdir -p ~/usys/riscv && cd ~/usys/riscv
git clone --recursive https://github.com/riscv/riscv-qemu.git
cd riscv-qemu
./configure \
    --target-list=riscv64-softmmu,riscv32-softmmu
make -j$(nproc)
make install
{% endhighlight %}

The instance of `qemu` we just built comes with [a few machines](https://github.com/riscv/riscv-qemu/wiki#machines) (specified via the `qemu-system-riscv32 -machine` option) ready to go, which is a nice convenience.

Next, let's install a RISC-V compatible copy of [OpenOCD](http://openocd.org/) and the RISC-V toolchain.

1. Download prebuilt versions of the RISC-V OpenOCD and the RISC-V toolchain from here: <https://www.sifive.com/boards>
2. Move and extract these files into the `~/usys/riscv` directory we created earlier.
{% highlight bash %}
cp openocd-<date>-<platform>.tar.gz ~/usys/riscv
cp riscv64-unknown-elf-gcc-<date>-<platform>.tar.gz ~/usys/riscv
cd ~/usys/riscv
tar -xvf openocd-<date>-<platform>.tar.gz
tar -xvf riscv64-unknown-elf-gcc-<date>-<platform>.tar.gz
{% endhighlight %}
{:start="3"}
3. Set the `RISCV_OPENOCD_PATH` and `RISCV_PATH` environment variables so other programs can find our toolchain.  This may look different depending on your OS and shell - I had to add these exports to my `~/.zshenv` file.  

{% highlight bash %}
# I put these two exports directly in my ~/.zshenv file - you may have to do something else.
export RISCV_OPENOCD_PATH="$HOME/usys/riscv/openocd-<date>-<version>"
export RISCV_PATH="$HOME/usys/riscv/riscv64-unknown-elf-gcc-<date>-<version>"
# Reload .zshenv with our new environment variables.  Restarting your shell will have a similar effect.
source ~/.zshenv
{% endhighlight %}

{:start="4"}
4. We'll also create a symbolic link into `/usr/local/bin` for this executable so that we can run it without specifying the full path to `~/usys/riscv/riscv64-unknown-elf-gcc-<date>-<version>/bin/riscv64-unknown-elf-gcc` whenever we want to use it.

{% highlight bash %}
# Symbolically link our gcc executable into /usr/local/bin.  Repeat this process for any other executables you want to quickly access.
ln -s ~/usys/riscv/riscv64-unknown-elf-gcc-8.2.0-<date>-<version>/bin/riscv64-unknown-elf-gcc /usr/local/bin
{% endhighlight %}

Et voilà, we have a working RISC-V toolchain!  All our executables, such as `riscv64-unknown-elf-gcc`, `riscv64-unknown-elf-gdb`, `riscv64-unknown-elf-ld`, etc, are located in `~/usys/riscv/riscv64-unknown-elf-gcc-<date>-<version>/bin/`.

### Hello, RISC-V!

Now that we have our toolchain setup, let's run an example RISC-V program.  I previously linked a SiFive repository called [freedom-e-sdk](https://github.com/sifive/freedom-e-sdk), which provides various programs we can try out.  Begin by recursively cloning this repository:

{% highlight bash %}
cd ~/wherever/you/want/to/clone/this
git clone --recursive https://github.com/sifive/freedom-e-sdk.git
cd freedom-e-sdk
{% endhighlight %}

And then compile all the example programs they provide:

{% highlight bash %}
make software
{% endhighlight %}

And we should (finally) be all set!  [As is tradition](https://stackoverflow.com/a/12785204), let's start with the "Hello, world" program provided by `freedom-e-sdk`.

{% highlight bash %}
07:38:47 PM twilcock ( master  | … )
~/projects/freedom-e-sdk
$ qemu-system-riscv32 -nographic -machine sifive_e -kernel software/hello/hello
Hello, World!
{% endhighlight %}

### What's next?

This is a great start, but my goal with these blog posts is to truly [shave the yak](https://seths.blog/2005/03/dont_shave_that/), and while we have confirmed that we have a working toolchain, there is a lot of magic hidden by the niceties of the `freedom-e-sdk` examples.  Note that we didn't have to set up any linker files or startup code - SiFive's provided board-support linker scripts, various Makefiles, and the [freedom-metal library](https://github.com/sifive/freedom-metal) take care of this for us.  

In part two of this series we'll break free from the `freedom-e-sdk` and make our own way.  We'll use `dtc` to examine the hardware layout of a `qemu` virtual machine, craft and examine a linker script, create a basic runtime to set up our stack, learn about some basic RISC-V assembly, and more.

The next post in this series has been posted, [click here to check it out.]({% post_url 2019-04-27-riscv-from-scratch-2 %})
