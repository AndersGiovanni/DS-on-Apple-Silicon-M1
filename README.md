# DS on Apple Silicon
How to setup your Apple Silicon MacBook for data science.

I personally had a hard time setting up my python projects on my new MacBook Air with M1 chip due to limited Apple Silicon support for many packages. And in order to spare you guys for the pain I've been through I decided to make a guide on how I did it.

As os March 25 2021, many python packages are still not supported for running natively on the Apple Silicon chips. Below I'll provide you with a guide on how to set up python and virtual environments in order to be able to work on data science. This will include both 'native' environments with optimized packages, but also Intel-based such that you can setup/run your 'normal' environments with the 'old' packages. 

## Installing Rosetta 2

Rosetta 2 can be installed by calling `softwareupdate --install-rosetta` in the terminal.

## Installing Xcode

- Install Xcode from AppStore
- Install CommandLine Tools: `xcode-select --install`
- Set CommandLine Tool version under Xcode --> Preferences --> Locations. Then select `Xcode 12.x`.
- Add SDKROOT Path to .zshrc file: `export SDKROOT="/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk"`

## Installing Homebrew

I have Homebrew installed for both the arm64 an x86_64 version. They can co-exist perfectly fine, though depending on your use-case, you might want one or the other to take precedence. 

### Install x86_64 Homebrew 

Homebrew x86_64 can be install by the command:

```bash
arch -x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```


### Install arm64 Homebrew

arm64 based Homebrew can be install by:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install brew dependencies

```bash
brew install libjpeg openblas openssl readline sqlite3 xz zlib
```

### Changing between arm64 Homebrew and x86_64 Homebrew 

Check which Homebrew has precedence by running `which brew`.
- **arm64**: `/opt/homebrew/bin/brew`
- **x86_64**: `/usr/local/bin/brew`

**Change to x86_64** by calling `eval $(/usr/local/bin/brew shellenv)`.

**Change to arm64** by calling `eval $(/opt/homebrew/bin/brew shellenv)`

**Making aliases** can be a smart way to call the desired brew. Add the following ot your `.zshrc` file:
```bash
alias ibrew='arch -x86_64 /usr/local/bin/brew'
alias mbrew='arch -arm64e /opt/homebrew/bin/brew'
```

## Install native python

Given that you have native Homebrew installed, you can install python 3.9.x, which is natively supported by the command:
`brew install python@3.9` or `mbrew install python@3.9` if you've made an alias. 

Ensure that your python installation is correct by `python3 -V`.

## Virtual environments

### Native virtual environments (poetry)

#### Install `pyenv`

```bash
curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
```

Add the following to your .zshrc file:

```bash
# pyenv
export PATH="$HOME/.pyenv/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi
```

TO BE CONTINUED

### Intel based virtual environments

Assuming you have x86_64 Homebrew installed with an `ibrew` alias (see Making aliases), you start by installing pyenv and some python build dependencies:

```bash
ibrew install pyenv bzip2 zlib
```

Now you can install your desired version of python - here I choose 3.8.5. Make sure that `which brew` is returning `/usr/local/bin/brew`, if not, then look at "Changing between arm64 Homebrew and x86_64 Homebrew".

```bash
CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include" LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" \
pyenv install --patch 3.8.5 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```

Now you can create a virtual environment

```bash
pyenv virtualenv 3.8.5 test_env
```

Activate virtual environment

```bash
pyenv test_env activate
```

Now make sure that your environment is installed correctly with the right python version by `python -V`.

From this activated environment you should be able to install packages like normal using `pip`. You should also be able to install and activate your `poetry` environment. **MAKE SURE THAT YOUR PYENV ENVIRONMENT IS ACTIVATED AT FIRST!!!**

## Credits

Below you can find all the resources I've used.

- [Install Rosetta 2.](https://osxdaily.com/2020/12/04/how-install-rosetta-2-apple-silicon-mac/)
- [Install x86_64 Homebrew.](https://stackoverflow.com/questions/65349982/issues-installing-python-3-x-via-pyenv)
- [Setting up virtual environments natively.](https://github.com/rybodiddly/Poetry-Pyenv-Homebrew-Numpy-TensorFlow-on-Apple-Silicon-M1)
- [Install Homebrew natively.](https://osxdaily.com/2021/02/06/installing-homebrew-apple-silicon-mac-native/)
- [Running Homebrew.](https://stackoverflow.com/questions/64882584/how-to-run-the-homebrew-installer-under-rosetta-2-on-m1-macbook/64883440#64883440)
- [Install native python](https://stackoverflow.com/questions/66037766/how-to-install-python-3-9-1-natively-support-to-apple-silicon)
- [Install x86_64 virtual environments](https://stackoverflow.com/questions/65349982/issues-installing-python-3-x-via-pyenv)
- [Making pyenv](https://medium.com/python-every-day/python-development-on-macos-with-pyenv-virtualenv-ec583b92934c)