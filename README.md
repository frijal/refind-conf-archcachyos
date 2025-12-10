# rEFInd Theme CachyOS Arch Linux
###### AUTHOR: diegons490 fork by me...

##### make sure, rEFInd packages was intalled in your system, if it is not installed, you can install using:
```shell
sudo pacman -S refind
```

##### After downloading the files, open the terminal in the folder where they are, and run the commands to install or uninstall the theme.
```shell
sudo bash install-refind-themes.sh
```
##### Themes will be set automatically, You can choose the resolution and size of the icons during installation.

# This is the content of installer script:
<details>
  <summary>Spoiler Alert: click to reveal... :)</summary>
  
```bash

#!/usr/bin/env bash
sudo refind-install
sudo refind-mkdefault
set -euo pipefail

# Colors and formatting
bold=$(tput bold)
normal=$(tput sgr0)
GREEN="\e[32;1m"
RED="\e[31;1m"
YELLOW="\e[33;1m"
CYAN="\e[36;1m"
RESET="\e[0m"

# Check root
if [[ $EUID -ne 0 ]]; then
    echo -e "${RED}${bold}This script must be run as root.${RESET}"
    exit 1
fi

# Set theme source directory
THEME_DIR=$(dirname "$(realpath "$0")")

# Function to find refind.conf
function find_refind_conf {
    echo -e "${YELLOW}Searching for refind.conf inside /boot (this may take a few seconds)...${RESET}"
    local found_path
    found_path=$(find /boot -type f -name refind.conf 2>/dev/null | head -n 1 || true)

    if [ -z "$found_path" ]; then
        echo -e "${RED}No refind.conf found under /boot.${RESET}"
        return 1
    else
        echo -e "${GREEN}refind.conf found at: ${found_path}${RESET}"
        while true; do
            read -rp "$(echo -e ${bold}Do you want to continue with this file? [Y/n]: ${normal})" answer
            case "$answer" in
                [Yy]*|"")
                    REFOUND_CONF_PATH="$found_path"
                    return 0
                    ;;
                [Nn]*)
                    return 2
                    ;;
                *)
                    echo -e "${RED}Please answer yes or no.${RESET}"
                    ;;
            esac
        done
    fi
}

# Function to ask for refind.conf path if needed
function ask_for_refind_conf {
    if ! find_refind_conf || [ "$?" -eq 2 ]; then
        echo -e "${YELLOW}Please enter the full path to your refind.conf file:${RESET}"
        read -rp "> " REFOUND_CONF_PATH
    fi
    while [[ ! -f "$REFOUND_CONF_PATH" ]]; do
        echo -e "${RED}${bold}File not found. Please enter a valid path to refind.conf:${RESET}"
        read -rp "> " REFOUND_CONF_PATH
    done
    if [[ ! -w "$REFOUND_CONF_PATH" ]]; then
        echo -e "${RED}${bold}You do not have write permission for $REFOUND_CONF_PATH${RESET}"
        exit 1
    fi
}

function install_theme {
    clear
    ask_for_refind_conf

    REFIND_DIR=$(dirname "$REFOUND_CONF_PATH")
    INSTALL_DIR="$REFIND_DIR/themes/cachy"

    # Remove old installation if exists
    if [ -d "$INSTALL_DIR" ]; then
        echo -e "${YELLOW}A previous installation was found at:${RESET} ${bold}$INSTALL_DIR${normal}"
        read -rp "$(echo -e ${bold}Remove it before continuing? [Y/n]: ${normal})" clean_ans
        case "$clean_ans" in
            [Yy]*|"")
                echo -e "${YELLOW}Removing...${RESET}"
                rm -rf "$INSTALL_DIR"
                ;;
            *)
                echo -e "${RED}Aborting to avoid overwriting files.${RESET}"
                exit 1
                ;;
        esac
    fi

    # Icon size
    echo -e "${CYAN}Pick an icon size:${RESET}"
    echo -e "${bold}1) Small       (128px - 80px)${normal}"
    echo -e "${bold}2) Medium      (256px - 160px)${normal}"
    echo -e "${bold}3) Large       (384px - 240px)${normal}"
    echo -e "${bold}4) Extra-large (512px - 320px)${normal}"
    read -rp "$(echo -e ${bold}Enter choice [1-4]: ${normal})" size_select
    size_select=${size_select:-1}
    case "$size_select" in
        1) size_big="128"; size_small="80" ;;
        2) size_big="256"; size_small="160" ;;
        3) size_big="384"; size_small="240" ;;
        4) size_big="512"; size_small="320" ;;
        *) echo -e "${RED}${bold}Invalid choice. Exiting.${RESET}"; exit 1 ;;
    esac

    echo -e "\nSelected size: ${GREEN}Big $size_big px, Small $size_small px${RESET}\n"

    # Resolution
    echo -e "${CYAN}Select monitor screen resolution:${RESET}"
    echo -e "${bold}1) 1280x720 pixel${normal}"
    echo -e "${bold}2) 1920x1080 pixel${normal}"
    echo -e "${bold}3) 2560x1440 pixel${normal}"
    echo -e "${bold}4) 3840x2160 pixel${normal}"
    echo -e "${bold}5) 3440x1440 pixel${normal}"
    echo -e "${bold}6) 5120x2160 pixel${normal}"
    read -rp "$(echo -e ${bold}Enter choice [1-6]: ${normal})" res_select
    res_select=${res_select:-2}
    case "$res_select" in
        1) res="1280x720" ;;
        2) res="1920x1080" ;;
        3) res="2560x1440" ;;
        4) res="3840x2160" ;;
        5) res="3440x1440" ;;
        6) res="5120x2160" ;;
        *) echo -e "${RED}${bold}Invalid resolution. Exiting.${RESET}"; exit 1 ;;
    esac
    res_width="${res%x*}"
    res_height="${res#*x}"
    echo -e "\nSelected resolution: ${GREEN}${res}${RESET}\n"

    echo -e "${CYAN}Installing theme...${RESET}"
    mkdir -p "$INSTALL_DIR/icons" "$INSTALL_DIR/background"
    cp -r "${THEME_DIR}/icons/"* "$INSTALL_DIR/icons/"

    bg_source="${THEME_DIR}/background/background-${res}.png"
    bg_target="${INSTALL_DIR}/background/background.png"
    [[ ! -f "$bg_source" ]] && echo -e "${RED}Background not found: $bg_source${RESET}" && exit 1
    cp "$bg_source" "$bg_target"

    # Generate config
    cat > "${INSTALL_DIR}/cachy.conf" << EOF
# Theme by diegons490
big_icon_size $size_big
small_icon_size $size_small
icons_dir themes/cachy/icons
selection_big themes/cachy/icons/selection-big.png
selection_small themes/cachy/icons/selection-small.png
banner themes/cachy/background/background.png
resolution $res_width $res_height
use_graphics_for linux,grub,osx,windows
timeout 10
EOF

    # Backup + edit refind.conf
    backup_path="${REFOUND_CONF_PATH}.bak.cachy-theme.$(date +%Y%m%d%H%M%S)"
    echo -e "${YELLOW}Backup: $backup_path${RESET}"
    cp "$REFOUND_CONF_PATH" "$backup_path"
    sed -i '/include themes\/cachy\/cachy.conf/d' "$REFOUND_CONF_PATH"
    echo -e "\n# Load rEFInd theme Cachy\ninclude themes/cachy/cachy.conf\n" >> "$REFOUND_CONF_PATH"

    echo -e "\n${GREEN}${bold}Installation complete!${RESET}"
    echo -e "Theme: ${bold}$INSTALL_DIR${normal}"
    echo -e "Modified: ${bold}$REFOUND_CONF_PATH${normal}"
    echo -e "Backup: ${bold}$backup_path${normal}\n"
}

function uninstall_theme {
    clear
    ask_for_refind_conf

    REFIND_DIR=$(dirname "$REFOUND_CONF_PATH")
    INSTALL_DIR="$REFIND_DIR/themes/cachy"

    local backup_file
    backup_file=$(ls -t "${REFOUND_CONF_PATH}".bak.cachy-theme.* 2>/dev/null | head -n 1 || true)

    if [[ ! -f "$backup_file" ]]; then
        echo -e "${RED}${bold}No backup found (${REFOUND_CONF_PATH}.bak.cachy-theme.*). Cannot proceed.${RESET}"
        exit 1
    fi

    echo -e "${YELLOW}Restoring backup: ${bold}$backup_file${normal}${RESET}"
    cp "$backup_file" "$REFOUND_CONF_PATH"

    echo -e "${YELLOW}Removing theme folder: ${bold}$INSTALL_DIR${normal}${RESET}"
    rm -rf "$INSTALL_DIR"

    echo -e "${YELLOW}Removing all backups created by this theme: ${REFOUND_CONF_PATH}.bak.cachy-theme.*${RESET}"
    rm -f "${REFOUND_CONF_PATH}".bak.cachy-theme.*

    echo -e "\n${GREEN}${bold}Theme removed and backups deleted!${RESET}"
}

function reconfigure_theme {
    clear
    ask_for_refind_conf

    REFIND_DIR=$(dirname "$REFOUND_CONF_PATH")
    INSTALL_DIR="$REFIND_DIR/themes/cachy"

    if [[ ! -f "$INSTALL_DIR/cachy.conf" ]]; then
        echo -e "${RED}${bold}No Cachy theme installation found at: $INSTALL_DIR${RESET}"
        exit 1
    fi

    echo -e "${CYAN}Cachy theme detected at: ${bold}$INSTALL_DIR${normal}${RESET}"

    echo -e "\n${CYAN}Pick a new icon size:${RESET}"
    echo -e "${bold}1) Small       (128px - 80px)${normal}"
    echo -e "${bold}2) Medium      (256px - 160px)${normal}"
    echo -e "${bold}3) Large       (384px - 240px)${normal}"
    echo -e "${bold}4) Extra-large (512px - 320px)${normal}"
    read -rp "$(echo -e ${bold}Enter choice [1-4]: ${normal})" size_select
    size_select=${size_select:-1}
    case "$size_select" in
        1) size_big="128"; size_small="80" ;;
        2) size_big="256"; size_small="160" ;;
        3) size_big="384"; size_small="240" ;;
        4) size_big="512"; size_small="320" ;;
        *) echo -e "${RED}Invalid choice. Exiting.${RESET}"; exit 1 ;;
    esac

    echo -e "\n${CYAN}Select new monitor resolution for background:${RESET}"
    echo -e "${bold}1) 1280x720 pixel${normal}"
    echo -e "${bold}2) 1920x1080 pixel${normal}"
    echo -e "${bold}3) 2560x1440 pixel${normal}"
    echo -e "${bold}4) 3840x2160 pixel${normal}"
    echo -e "${bold}5) 3440x1440 pixel${normal}"
    echo -e "${bold}6) 5120x2160 pixel${normal}"
    read -rp "$(echo -e ${bold}Enter choice [1-6]: ${normal})" res_select
    res_select=${res_select:-2}
    case "$res_select" in
        1) res="1280x720" ;;
        2) res="1920x1080" ;;
        3) res="2560x1440" ;;
        4) res="3840x2160" ;;
        5) res="3440x1440" ;;
        6) res="5120x2160" ;;
        *) echo -e "${RED}Invalid resolution. Exiting.${RESET}"; exit 1 ;;
    esac
    res_width="${res%x*}"
    res_height="${res#*x}"

    bg_source="${THEME_DIR}/background/background-${res}.png"
    bg_target="${INSTALL_DIR}/background/background.png"

    if [[ ! -f "$bg_source" ]]; then
        echo -e "${RED}Background not found: $bg_source${RESET}"
        exit 1
    fi

    cp "$bg_source" "$bg_target"

    echo -e "${YELLOW}Updating cachy.conf...${RESET}"

    cat > "${INSTALL_DIR}/cachy.conf" << EOF
# Theme by diegons490
big_icon_size $size_big
small_icon_size $size_small
icons_dir themes/cachy/icons
selection_big themes/cachy/icons/selection-big.png
selection_small themes/cachy/icons/selection-small.png
banner themes/cachy/background/background.png
resolution $res_width $res_height
use_graphics_for linux,grub,osx,windows
timeout 10
EOF

    echo -e "\n${GREEN}${bold}Reconfiguration complete!${RESET}"
    echo -e "Updated resolution: ${bold}${res}${normal}"
    echo -e "Icon sizes: ${bold}Big $size_big px, Small $size_small px${normal}\n"
}

# Main menu
clear
echo -e "${bold}${CYAN}######################################${RESET}"
echo -e "${bold}${CYAN}### rEFInd CachyOS Theme Installer ###${RESET}"
echo -e "${bold}${CYAN}######################################${RESET}"
echo
echo -e "${bold}1) Install theme"
echo -e "2) Remove theme and restore backup"
echo -e "3) Reconfigure resolution and icon size"
echo -e "0) Cancel${normal}"
echo
read -rp "$(echo -e ${bold}Choose an option [0-3]: ${normal})" menu_choice

case "$menu_choice" in
    1) install_theme ;;
    2) uninstall_theme ;;
    3) reconfigure_theme ;;
    0) echo -e "${YELLOW}Cancelled by user.${RESET}" && exit 0 ;;
    *) echo -e "${RED}Invalid option. Exiting.${RESET}" && exit 1 ;;
esac
```
</details>

# Screenshot
<div align=center>

  ![screenshot](/preview.png?raw=true)
![screenshot](/rEFInd-active.jpeg?raw=true)

</div>
