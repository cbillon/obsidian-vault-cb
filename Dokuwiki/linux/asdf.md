---
tags:
  - tools
  - asdf
---

====== Installation ASDF ======

[[https://www.digitalocean.com/community/tutorials/how-to-install-asdf-to-manage-multiple-programming-language-runtime-versions-on-ubuntu-22-04]]

asdf relies on the installation of a core that, alone, does not have functionality. The asdf core relies on separate plugins that are specific to a given programming language or program. Most commonly, it is used to install and manage multiple versions of a given programming language. It’s recommended that you download the asdf core with git, which comes installed with Ubuntu 22.04. To get the latest version of asdf, clone the latest branch from the asdf repository:

  git clone https://github.com/asdf-vm/asdf.git ~/.asdf 
  
asdf requires a unique installation depending on the combination of shell type and method it was downloaded. By default, Ubuntu uses Bash for its shell, which uses the ~/.bashrc file for configuration and customization. To enable the usage of the asdf command, you will have to add the following line:

  echo ". $HOME/.asdf/asdf.sh" >> ~/.bashrc
  
Next, make sure your changes are applied to your current session:

  source ~/.bashrc


