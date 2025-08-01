# pyenv (Python Virtual Environment)

## Installation

```bash
sudo pacman -S pyenv
yay -S pyenv-virtualenv
```

Then

add the following to fish config file:

```fish
# Load pyenv into the shell
status --is-interactive; and pyenv init --path | source
status --is-interactive; and pyenv init - | source
status --is-interactive; and pyenv virtualenv-init - | source
```

## Usage

### Create a new virtual environment

```bash
# Install a specific Python version if not already installed
pyenv install <python_version>

# Create a new virtual environment
pyenv virtualenv <python_version> <env_name>

# Activate the virtual Environment
pyenv activate <env_name>
```
