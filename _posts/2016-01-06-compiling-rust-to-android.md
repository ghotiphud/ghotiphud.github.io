---
title:  "Compiling Rust to an Android Target"
categories: rust android cross-compiling
---

## Multirust - Use It

This post started life as an exploration in cross-compiling, but at this point it's become more of an intro to using [multirust-rs][1] and [cargo][2] which simplify things considerably.  So, we'll get to the fun Android parts, but for now stick with me.

The first thing to do if you haven't installed multirust is to go get that done ([multirust-rs][1]), see you back here in a minute.

.

.

.

Go ahead and start running `multirust update`.  Depending on your connection speed this step may take a while.  This will download the **stable**, **beta**, and **nightly** compilers from http://static.rust-lang.org/dist.  We'll need to keep this URL in mind for a bit as we'll see it pop up again later.

The next thing we'll want to do is setup our default toolchain `multirust default stable`.  I choose to use **stable** by default as that seems like a decent default.  Might as well stick with the blessed compiler version (We'll be adding an override later because we will need nightly for cross-compiling android).

So, now that we have that out of the way, we can move on to running something.

## New Crate

```bash
$ cargo new --bin android-test
$ cd android-test
```

Here we've set up our new executable crate that we're targetting for android, but honestly, who wants to develop all their logic on an emulator or phone hardware fiddling with `adb` the whole time. I know I don't. IDE integration would make this less of a pain, but things are still much slower to test and cycle times are directly linked to productivity, so let's be efficient.

At this point you should be able to run `cargo run` and see a nice `Hello world!`. Beautiful!

## Setting up the Cross-Compiler

I'm going to cheat a bit here, and only compile for ARM versions of android as opposed to atom x86, x64 versions.  But hey, the majority of devices are ARM, Servo doesn't even support this yet, and I don't have an atom device to test on.  Maybe I'll update this later if I end up testing this further...

Onward!

It's lucky for us that rust is build on top of LLVM, which is a cross-compiler by default.  This will make things fairly smooth, but we haven't needed to specify which architecture we're compiling for.  `cargo build/run/test` by default will pick your current architecture as the target.

`cargo build` is equivalent to `cargo build --target=x86_64-unknown-linux-gnu` on my machine.  Your experience may vary.

**INFO** - **arm-linux-androideabi** is the target for android ARM devices.

You may be thinking that you can just `cargo build --target=arm-linux-androideabi` and away you go, but hold your horses. It would be awesome if that worked, it's not quite *that* smooth.

`$ cargo build --target=arm-linux-androideabi --verbose`

Well, that gives us a nice error message.

```
Process didn't exit successfully: `rustc - --crate-name _ --crate-type dylib --crate-type bin --print=file-names --target arm-linux-androideabi` (exit code: 101)
--- stderr
error: Error loading target specification: Could not find specification for target "arm-linux-androideabi"
```

We need a "specification for target 'arm-linux-androideabi'". Where can we get one of those?  After consulting the very friendly experts on the #rust IRC channel I found out that it meant we needed a stdlib compiled to target arm-linux-androideabi. Well, we could build the rust standard library for arm-linux-androideabi with 

```bash
$ ./configure --target=arm-unknown-linux-gnueabihf,x86_64-unknown-linux-gnu
$ make -j$(nproc)
$ sudo make install
```

But that would take ~1 hr of compilation time... I lack that amount of patience... More details [here][3] if you're trying to kill some time or heat your house or something.

Remember when I mentioned http://static.rust-lang.org/dist earlier? Well, a bit of sleuthing there and you'll find that each dated folder not only contains rust tarballs for your desktop architecture, but the kindly build gnomes have produced android std libs as well!  Conveniently, exactly the thing we need.

There is one caveat as of this writing that the android builds seem to be nightlies only.  But that's okay.  As long as we match our compiler version with the android std version we'll be fine.  

I've picked the date **2015-12-14** to base my toolchain, but mostly likely you'll use whatever the newest nightly build is at the time you do this.

**NOTE**: Not all nightlies are built equally... While updating this post, I tried **2016-01-05** which failed to compile rustc-serialize.  Buyer beware!

Without further ado, multirust makes this a simple thing to set up.

```bash
$ multirust override nightly-2015-12-14 
```

Now download http://static.rust-lang.org/dist/2015-12-14/rust-std-nightly-arm-linux-androideabi.tar.gz and unzip it.  You'll find a folder **rust-std-nightly-arm-linux-androideabi/rust-std-arm-linux-androideabi/lib/rustlib/arm-linux-androideabi** which you can copy to **~/.multirust/toolchains/nightly-2015-12-14/lib/rustlib**

Congrats! You now have a cross-compiler!

## Linker Error!

Let's try that compilation again for android.

```bash
$ cargo build --target=arm-linux-androideabi --verbose

Compiling android-test v0.1.0 (file:///mnt/storage/dev/android-test)
    Running `rustc src/main.rs --crate-name android_test --crate-type bin -g --out-dir /mnt/storage/dev/android-test/target/arm-linux-androideabi/debug --emit=dep-info,link --target arm-linux-androideabi -L dependency=/mnt/storage/dev/android-test/target/arm-linux-androideabi/debug -L dependency=/mnt/storage/dev/android-test/target/arm-linux-androideabi/debug/deps`
error: linking with `cc` failed: exit code: 1
...
```

So, our compiler worked, but now our linker is making things difficult... Luckily there's [android-glue][4] which contains **apk-builder** which we will substitute for our linker.  

```bash
# Create the .cargo/config file
$ mkdir -p .cargo
$ echo '[target.arm-linux-androideabi]
linker = "apk-builder/apk-builder/target/release/apk-builder"' > .cargo/config

# Add the apk-builder
$ git clone https://github.com/tomaka/android-rs-glue apk-builder
$ cd apk-builder/apk-builder
$ cargo build --release
$ cd ../..
```

apk-builder depends upon the Android SDK and NDK, so we'll need those as well.

## Setting up your Android Toolchain (See [Servo Wiki][5])

First, we need some android development tools like the NDK, SDK, and toolchain, for cross-compiling:

* The Android NDK can be downloaded from http://developer.android.com/ndk/downloads/index.html
* The Android SDK can be downloaded from http://developer.android.com/sdk/index.html

To install the NDK and SDK, refer to the guidelines on the website. When you install the SDK, run the UI tool at `tools/android` and update the SDK in order to install a full set of Android libraries. You may use the Android-18 platform. From the commandline, you may also do `tools/android - update sdk --no-ui`.

First, create an Android toolchain from the NDK. Example command to setup standalone Android tool chain:
`'ndk root'/build/tools/make-standalone-toolchain.sh --platform="android-18" --toolchain=arm-linux-androideabi-4.8 --install-dir='dir to install' --ndk-dir='ndk dir' --arch=arm`

Set the following environment variables while building. (You may want to export them from a configuration file like `.bashrc`.)

```bash
ANDROID_SDK="/path/to/sdk"
ANDROID_NDK="/path/to/ndk"
ANDROID_TOOLCHAIN="/path/to/toolchain"
PATH="$PATH:$ANDROID_TOOLCHAIN/bin" # add the toolchain to your $PATH
```

If you're on Debian (not Ubuntu), you may need to install the packages below.

```bash
sudo apt-get install curl ia32-libs ant
```

If you're on Ubuntu, you may need to do:

```bash
sudo apt-get install curl libc6:i386 ant lib32z1 openjdk-7-jdk
```

If you're on OSX, install these packages:

```bash
brew install nspr ant
```

## Cross-Compiling for the Android EABI

NOTE: I've added the ANDROID_* variables to my `.bashrc`, so the following commands assume you've done the same.

We are ready! Here we go! 

```bash
$ ANDROID_HOME=$ANDROID_SDK NDK_HOME=$ANDROID_NDK NDK_STANDALONE=$ANDROID_TOOLCHAIN cargo build --target=arm-linux-androideabi
   Compiling android-test v0.1.0 (file:///mnt/storage/dev/android-test)
```

Kind of anti-climactic...

## Testing If It Actually Worked

So, we have an apk, but it doesn't actually do anything... So, in the tradition of typical programming examples we will be writing "Hello world!".

First, add a dependency to android_glue in Cargo.toml:

```rust
[target.arm-linux-androideabi.dependencies.android_glue]
version = "^0.1"
```

Then, add extern crate android_glue and invoke android_start! in your main crate.

```rust
#[cfg(target_os = "android")]
#[macro_use] 
extern crate android_glue;

#[cfg(target_os = "android")]
android_start!(main);

fn main() {
    println!("Hello world from Rust!");
}
```

We'll recompile, and deploy to an emulator:

```bash
$ ANDROID_HOME=$ANDROID_SDK NDK_HOME=$ANDROID_NDK NDK_STANDALONE=$ANDROID_TOOLCHAIN cargo build --target=arm-linux-androideabi --release

# Need the .apk extension or adb will complain
$ cd target/arm-linux-androideabi/release
$ mv android-test android-test.apk
$ adb install -r android-test.apk

# ~~~ Run the app by launching in the emulator here ~~~

$ adb logcat
```

You should see a few lines similar to:

```
01-06 11:14:56.321  2096  2111 D RustAndroidGlueStdouterr: Entering android_main
01-06 11:14:56.324  2096  2111 D RustAndroidGlueStdouterr: created application thread
01-06 11:14:56.329  2096  2112 D RustAndroidGlueStdouterr: Hello world from Rust!
```

## It's Working!!!!!

Thanks for sticking with me this far because this got much longer than intended.  I may continue investigating further, but without any immediate use for this work it might hang out there a while.  We shall see.  

As a side note, I'm really enjoying exploring Rust.  The community is great, and for a first foray into systems programming coming from a .Net/C# focussed day job it has been remarkably smooth.  I recommend taking the leap for anyone who has the inclination.  Farewell for now.

[1]: https://github.com/Diggsey/multirust-rs
[2]: https://crates.io/
[3]: https://github.com/japaric/ruststrap/blob/master/1-how-to-cross-compile.md
[4]: https://github.com/tomaka/android-rs-glue
[5]: https://github.com/servo/servo/wiki/Building-for-Android