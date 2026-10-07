# ROS 2 sobre Android (Emulado) - Ambiente de Desenvolvimento

Este repositório contém os passos para configurar um ambiente ROS 2 Humble rodando dentro de um Android emulado, usando Android Studio Emulator (AVD), Termux e proot-distro.

## Prerequisites

- **Operating System**: Ubuntu 22.04 LTS
- **Ferramenta de Emulação**: Android Studio (Android Emulator / AVD)
- **Dispositivo Virtual**: Pixel 6
- **Imagem de Sistema**: Google APIs, x86_64, API 34 "UpsideDownCake" (Android 14.0)
- **ROS 2**: Humble 

## Installation

#### Android Studio & Virtual Device (AVD)

* Verifique o suporte à virtualização e instale o Android Studio:
  ```bash
  sudo apt install cpu-checker -y
  kvm-ok
  sudo snap install android-studio --classic
  ```
* Na primeira abertura, mantenha **"Standard"** marcado e clique em **Next**. Aceite as licenças (**SDK license**) e clique em **Finish**.
* Crie o dispositivo virtual em **More Actions → Virtual Device Manager → + (Create Virtual Device)**:
  1. Escolha o perfil de telefone **Pixel 6**
  2. Na imagem de sistema, selecione **API 34 "UpsideDownCake"; Android 14.0**
  3. Em **Services**, escolha **"Google APIs"**
  4. **Finish**

#### Termux + Termux:API

* Via terminal do Ubuntu (Fora do Emulador) - Baixe os APKs mais recentes direto do GitHub :
  ```bash
  sudo apt install curl
  TERMUX_URL=$(curl -s https://api.github.com/repos/termux/termux-app/releases/latest \
    | grep "browser_download_url.*universal.apk" | cut -d '"' -f4)
  wget "$TERMUX_URL" -O termux-app.apk

  TERMUX_API_URL=$(curl -s https://api.github.com/repos/termux/termux-api/releases/latest \
    | grep "browser_download_url.*apk" | cut -d '"' -f4)
  wget "$TERMUX_API_URL" -O termux-api.apk
  ```
* Instale via `adb`:
  ```bash
  sudo apt install adb
  adb install termux-app.apk
  adb install termux-api.apk
  ```
  Se aparecer `Performing Streamed Install` / `Success` para os dois, deu certo.
* No emulador, abra a gaveta de apps e inicie o **Termux** (ícone preto `>_`). Espere a instalação do bootstrap e confirme se aparece um prompt `$` no final, sem popup de erro.

#### Ubuntu 22.04 (proot-distro)

* Dentro do Termux:
  ```bash
  pkg update -y
  pkg install proot-distro termux-api -y

  proot-distro install ubuntu:22.04
  proot-distro login ubuntu
  ```
  O prompt deve mudar para `root@localhost:~#`, confirmando que você está dentro do Ubuntu 22.04.

#### ROS 2 Humble

* Já dentro do Ubuntu:
  ```bash
  apt update && apt install curl gnupg2 lsb-release locales -y
  locale-gen en_US.UTF-8
  curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
    -o /usr/share/keyrings/ros-archive-keyring.gpg
  echo "deb [signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
    http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" \
    > /etc/apt/sources.list.d/ros2.list
  apt update
  apt install ros-humble-ros-base -y
  source /opt/ros/humble/setup.bash
  ```
  * Durante o `apt install`, pode aparecer a configuração de fuso horário do pacote `tzdata` (é interativa): digite o número da região (ex.: `2` para América) e depois selecione o país e a cidade/fuso (ex.: Brasil, Recife UTC-3).
  
  Confirme que com o `lsb_release -cs` se retorna **"jammy"**.
   * Rode novamente:
    ```bash
    apt install ros-humble-ros-base -y
    ```
* Instale os nós de demonstração (não incluídos no `ros-base`):
  ```bash
  apt install ros-humble-demo-nodes-cpp -y
  ```

---

## Getting started

* **Ativar o ROS 2 e listar os tópicos.**
  ```bash
  source /opt/ros/humble/setup.bash
  ros2 topic list
  ```
  Se aparecer uma lista (tipo `/parameter_events` e `/rosout`) sem erro, o ROS 2 Humble está funcionando.
