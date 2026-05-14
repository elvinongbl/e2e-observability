```bash
# Switch to root
sudo su

# Ubuntu 24.04.4
# Software packages for pyenv to build Python from source
apt install -y python-is-python3
apt install -y build-essential gdb lcov pkg-config \
               libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev \
               liblzma-dev libncurses-dev libreadline-dev \
               libsqlite3-dev libssl-dev \
               lzma lzma-dev tk-dev uuid-dev zlib1g-dev

# Install pyenv
curl https://pyenv.run | bash

# Add pyenv environment to bottom of /root/.bashrc
nano /root/.bashrc
---
export PYENV_ROOT="$HOME/.pyenv"
if [ -d $HOME/.pyenv/bin ]; then
    export PATH=${PYENV_ROOT}/bin:$PATH
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
fi
---

# Activate pyenv
source /root/.bashrc

# Test pyenv
pyenv versions

# Install specific Python version
# Note: 3.10.19 is the version that I tested with NPU monitoring tool
# Note: The installation will take a while (python compilation)
# Note: The default pyenv folder is installed under /root/.pyenv
pyenv install 3.10.19

# Set global python version for root
pyenv global 3.10.19
```
