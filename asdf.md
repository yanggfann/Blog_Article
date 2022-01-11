# asdf

[asdf](https://asdf-vm.com/guide/introduction.html) is a **tool version manager**. All tool version definitions are contained within one file (**.tool-versions**) which you can check in to your project's Git repository to share with your team, ensuring everyone is using the exact same versions of tools.

## Why use asdf?

asdf ensures teams are using the exact same versions of tools, with support for many tools via a plugin system, and the simplicity and familiarity of being a single Shell script you include in your Shell config.

asdf is not intended to be a system package manager. It is a tool version manager. 

## [nvm VS asdf](https://www.libhunt.com/compare-nvm-vs-asdf)

### nvm

Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions (by nvm-sh)

### asdf

Extendable version manager with support for Ruby, Node.js, java, Elixir, Erlang & more (by asdf-vm)

## Use asdf

### Installing dependencies

Homebrew is a Package Manager. Dependencies will be automatically installed by Homebrew.

### Downloading asdf core

```
brew install asdf
```

### Installing asdf

ZSH & Homebrew

Add asdf.sh to your ~/.zshrc with:

```
echo -e "\n. $(brew --prefix asdf)/libexec/asdf.sh" >> ${ZDOTDIR:-~}/.zshrc
```

This completes the installation of the asdf core 🎉. asdf is only useful once you install a **plugin**, install a **tool** and manage its **versions**.

### Installing a plugin for each tool/runtime you wish to manage

```
asdf plugin-add java https://github.com/halcyon/asdf-java.git
```

### Installing a version of the tool/runtime

```
asdf install java openjdk-17.0.1
```

### Setting global and project versions via .tool-versions config files

- Global

```
asdf global java openjdk-17.0.1
```

- Local

```
asdf local java openjdk-11
```

## Using Existing Tool Version Files

[asdf-nodejs](https://github.com/asdf-vm/asdf-nodejs/) supports this via both .nvmrc and .node-version files. To enable this, add the following to your asdf configuration file $HOME/.asdfrc:

```
legacy_version_file = yes
```

See the [configuration](https://asdf-vm.com/manage/configuration.html#environment-variables) reference page for more config options.

You can replace nvm with asdf to manage NodeJS versions. Refer to: https://faizmokhtar.com/posts/migrating-from-nvm-to-asdf/

## asdf --help

You can achieve the more information of asdf command when running the `asdf --help` command in the terminal.

**asdf current <name>**                     Display current version set or being used for package

**asdf list <name>**                        List installed versions of a package

**asdf list all <name> [<version>]**        List all versions of a package and optionally filter the returned versions

**asdf plugin list all**                    List plugins registered on asdf-plugins repository with URLs

**asdf install <name> <version>**           Install a specific version of a package
