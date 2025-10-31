# ♥ X-1-Terminal-Menu ♥

​X-1 System Diagnostics:

Advanced, color-coded Bash menu for managing Linux/Termux environments.

Features:
SSH, Security, Notes, and Live System Monitor.


<img src="IMG_20251031_041142.jpg" width="400"/>


 ♦Removing Termux welcome message♦ (optional):


 $ - rm $PREFIX/etc/motd
 
 # nano
 $-pkg install nano

  # Copy script
   $- nano ~/.bashrc
   # Paste the scrip
   CTRL + o Enter CTRL X 
   Exit Termux completely and restart.

 #!/bin/bash

# =========== COLORS ============
RED=$(printf '\033[0;31m')
YELLOW=$(printf '\033[1;33m')
GREEN=$(printf '\033[0;32m')
CYAN=$(printf '\033[0;36m')
BLUE=$(printf '\033[0;34m')
MAGENTA=$(printf '\033[0;35m')
RESET=$(printf '\033[0m')

# =========== CONFIG FILES ============
CONFIG_DIR="$HOME/.x1_config"
NOTES_FILE="$CONFIG_DIR/notes.txt"
SSH_FILE="$CONFIG_DIR/ssh_hosts.txt"
SHORTCUTS_FILE="$CONFIG_DIR/shortcuts.txt"
THEME_FILE="$CONFIG_DIR/theme.conf"
PASSWORD_FILE="$CONFIG_DIR/.passwd"

# Create config directory
mkdir -p "$CONFIG_DIR"

# =========== THEME SYSTEM ============
load_theme() {
  if [[ -f "$THEME_FILE" ]]; then
    source "$THEME_FILE"
  else
    # Default theme
    THEME_PRIMARY=$GREEN
    THEME_SECONDARY=$CYAN
    THEME_ACCENT=$YELLOW
    save_theme
  fi
}

save_theme() {
  cat > "$THEME_FILE" << EOF
THEME_PRIMARY='$THEME_PRIMARY'
THEME_SECONDARY='$THEME_SECONDARY'
THEME_ACCENT='$THEME_ACCENT'
EOF
}

# =========== PASSWORD SYSTEM ============
check_password() {
  if [[ -f "$PASSWORD_FILE" ]]; then
    echo -e "${YELLOW}╔════════════════════════════════════════╗${RESET}"
    echo -e "${YELLOW}║          PASSWORD REQUIRED             ║${RESET}"
    echo -e "${YELLOW}╚════════════════════════════════════════╝${RESET}"
    printf "${CYAN}Enter password: ${RESET}"
    read -s pass
    echo ""
    stored_pass=$(cat "$PASSWORD_FILE")
    if [[ "$pass" != "$stored_pass" ]]; then
      echo -e "${RED}Access Denied!${RESET}"
      sleep 2
      exit 1
    fi
    echo -e "${GREEN}Access Granted!${RESET}"
    sleep 1
  fi
}

# =========== LIVE CLOCK ============
show_live_clock() {
  tput cup 0 60
  echo -e "${CYAN}$(date '+%H:%M:%S')${RESET}"
}

# =========== MATRIX EFFECT ============
matrix_effect() {
  clear
  echo -e "${GREEN}"
  for i in {1..30}; do
    echo -n "$(shuf -i 0-1 -n 80 | tr -d '\n')"
    echo ""
    sleep 0.05
  done
  echo -e "${RESET}"
  sleep 0.5
}

# =========== MAIN MENU LOOP ============
main_menu() {
  load_theme
  
  while true; do
    clear
    
    # =========== X-1 BANNER - BIG AND CENTERED ============
    echo -e "${THEME_PRIMARY}"
    echo ""
    echo "    ██╗  ██╗      ██╗"
    echo "    ╚██╗██╔╝      ╚═╝"
    echo "     ╚███╔╝       ██╗"
    echo "     ██╔██╗       ╚═╝"
    echo "    ██╔╝ ██╗      ██╗"
    echo "    ╚═╝  ╚═╝      ╚═╝"
    echo -e "${RESET}"
    echo -e "${THEME_SECONDARY}      Created by X-616 ^∞^${RESET}"
    echo ""

    # =========== SYSTEM INFO - HACKER STYLE ============
    echo -e "${THEME_PRIMARY}╔════════════════════════════════════════╗${RESET}"
    echo -e "${THEME_PRIMARY}║      SYSTEM DIAGNOSTICS - X-1         ║${RESET}"
    echo -e "${THEME_PRIMARY}╚════════════════════════════════════════╝${RESET}"
    
    # TIME & HOST
    now=$(date "+%Y-%m-%d %H:%M:%S")
    host=$(hostname)
    user=$(whoami)
    ip=$(ip -o -4 addr show wlan0 2>/dev/null | awk '{print $4}' | cut -d/ -f1)
    : "${ip:=N/A}"
    
    printf "${THEME_SECONDARY}[TIME]${RESET}    ${THEME_PRIMARY}%s${RESET}\n" "$now"
    printf "${THEME_SECONDARY}[USER]${RESET}    ${THEME_PRIMARY}%s@%s${RESET}\n" "$user" "$host"
    printf "${THEME_SECONDARY}[IP]${RESET}      ${THEME_PRIMARY}%s${RESET}\n\n" "$ip"

    # RAM usage
    ram_total=$(free -m 2>/dev/null | awk '/^Mem:/ {print $2}')
    ram_used=$(free -m 2>/dev/null | awk '/^Mem:/ {print $3}')
    ram_perc=$(( ram_used * 100 / ram_total ))

    # CPU usage
    cpu=$(top -bn1 2>/dev/null | grep "Cpu(s)" | awk '{print $2 + $4}' || echo "0")
    cpu_perc=${cpu%.*}

    # Battery
    batt="N/A"
    if command -v termux-battery-status >/dev/null 2>&1; then
      batt=$(termux-battery-status 2>/dev/null | grep percentage | grep -o '[0-9]\+')
    fi

    # Disk
    disk_info=$(df -h / 2>/dev/null | awk 'NR==2 {print $3 "/" $2 " [" $5 "]"}')

    # Display in hacker style with numbers
    printf "${THEME_PRIMARY}╔════════════════════════════════════════╗${RESET}\n"
    printf "${THEME_PRIMARY}║  RAM:${RESET}     ${THEME_SECONDARY}%04d${RESET}/${THEME_PRIMARY}%04d${RESET} MB  ${THEME_ACCENT}[%03d%%]${RESET}    ${THEME_PRIMARY}║${RESET}\n" "$ram_used" "$ram_total" "$ram_perc"
    printf "${THEME_PRIMARY}║  CPU:${RESET}     ${THEME_SECONDARY}%03d%%${RESET}                        ${THEME_PRIMARY}║${RESET}\n" "$cpu_perc"
    printf "${THEME_PRIMARY}║  BAT:${RESET}     ${THEME_SECONDARY}%s%%${RESET}                         ${THEME_PRIMARY}║${RESET}\n" "$batt"
    printf "${THEME_PRIMARY}║  DISK:${RESET}    ${THEME_SECONDARY}%-22s${RESET}  ${THEME_PRIMARY}║${RESET}\n" "$disk_info"
    printf "${THEME_PRIMARY}╚════════════════════════════════════════╝${RESET}\n\n"

    # =========== MAIN MENU OPTIONS ============
    echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
    echo -e "${THEME_ACCENT}║           MAIN MENU                    ║${RESET}"
    echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}"
    echo -e "${THEME_PRIMARY}[1]${RESET}  Browser        ${THEME_PRIMARY}[2]${RESET}  Terminal"
    echo -e "${THEME_PRIMARY}[3]${RESET}  System Update  ${THEME_PRIMARY}[4]${RESET}  Network Tools"
    echo -e "${THEME_PRIMARY}[5]${RESET}  File Manager   ${THEME_PRIMARY}[6]${RESET}  Live Monitor"
    echo -e "${THEME_PRIMARY}[7]${RESET}  Notes/TODO     ${THEME_PRIMARY}[8]${RESET}  Shortcuts"
    echo -e "${THEME_PRIMARY}[9]${RESET}  SSH Manager    ${THEME_PRIMARY}[10]${RESET} Downloads"
    echo -e "${THEME_PRIMARY}[11]${RESET} Security       ${THEME_PRIMARY}[12]${RESET} Backup"
    echo -e "${THEME_PRIMARY}[13]${RESET} Speedtest      ${THEME_PRIMARY}[14]${RESET} Themes"
    echo -e "${THEME_PRIMARY}[15]${RESET} Matrix Effect  ${THEME_PRIMARY}[0]${RESET}  Exit\n"
    
    printf "${THEME_SECONDARY}X-1>${RESET} "
    read -r choice

    case $choice in
      1) browser_menu ;;
      2) terminal_mode ;;
      3) system_update ;;
      4) network_tools ;;
      5) file_manager ;;
      6) live_monitor ;;
      7) notes_manager ;;
      8) shortcuts_manager ;;
      9) ssh_manager ;;
      10) download_manager ;;
      11) security_menu ;;
      12) backup_system ;;
      13) speedtest_run ;;
      14) theme_selector ;;
      15) matrix_effect ;;
      0) exit_program ;;
      *) echo -e "${RED}Invalid option!${RESET}"; sleep 1 ;;
    esac
  done
}

# =========== BROWSER MENU ============
browser_menu() {
  echo -e "\n${THEME_ACCENT}Opening default browser...${RESET}"
  if command -v termux-open-url >/dev/null 2>&1; then
    printf "${THEME_SECONDARY}Enter URL (or press Enter for google.com):${RESET} "
    read -r url
    : "${url:=https://www.google.com}"
    termux-open-url "$url"
  elif command -v xdg-open >/dev/null 2>&1; then
    printf "${THEME_SECONDARY}Enter URL (or press Enter for google.com):${RESET} "
    read -r url
    : "${url:=https://www.google.com}"
    xdg-open "$url" &
  else
    echo -e "${RED}Error: No browser command found${RESET}"
  fi
  sleep 2
}

# =========== TERMINAL MODE ============
terminal_mode() {
  echo -e "\n${THEME_PRIMARY}Entering Terminal Mode...${RESET}"
  sleep 1
  export PS1="${THEME_PRIMARY}X-1_HACK# ${RESET}"
  bash --norc
}

# =========== SYSTEM UPDATE ============
system_update() {
  echo -e "\n${THEME_ACCENT}Starting System Update...${RESET}"
  sleep 1
  
  if command -v pkg >/dev/null 2>&1; then
    echo -e "${THEME_SECONDARY}Updating package lists...${RESET}"
    pkg update -y
    echo -e "${THEME_SECONDARY}Upgrading packages...${RESET}"
    pkg upgrade -y
  elif command -v apt >/dev/null 2>&1; then
    echo -e "${THEME_SECONDARY}Updating package lists...${RESET}"
    sudo apt update
    echo -e "${THEME_SECONDARY}Upgrading packages...${RESET}"
    sudo apt upgrade -y
  else
    echo -e "${RED}Error: No package manager found${RESET}"
  fi
  
  echo -e "\n${THEME_PRIMARY}Update complete!${RESET}"
  sleep 3
}

# =========== NETWORK TOOLS ============
network_tools() {
  clear
  echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
  echo -e "${THEME_ACCENT}║         NETWORK SCANNER                ║${RESET}"
  echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
  
  echo -e "${THEME_PRIMARY}[1]${RESET} Show IP Info"
  echo -e "${THEME_PRIMARY}[2]${RESET} Ping Test"
  echo -e "${THEME_PRIMARY}[3]${RESET} Port Scanner (requires nmap)"
  echo -e "${THEME_PRIMARY}[0]${RESET} Back\n"
  
  printf "${THEME_SECONDARY}Choice>${RESET} "
  read -r net_choice
  
  case $net_choice in
    1)
      echo -e "\n${THEME_SECONDARY}=== IP Information ===${RESET}"
      ip addr show 2>/dev/null || ifconfig
      ;;
    2)
      printf "\n${THEME_SECONDARY}Enter host to ping:${RESET} "
      read -r ping_host
      ping -c 4 "$ping_host"
      ;;
    3)
      if command -v nmap >/dev/null 2>&1; then
        printf "\n${THEME_SECONDARY}Enter IP to scan:${RESET} "
        read -r scan_ip
        nmap "$scan_ip"
      else
        echo -e "${RED}nmap not installed. Install with: pkg install nmap${RESET}"
      fi
      ;;
  esac
  
  echo -e "\n${THEME_ACCENT}Press Enter to continue...${RESET}"
  read
}

# =========== FILE MANAGER ============
file_manager() {
  while true; do
    clear
    echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
    echo -e "${THEME_ACCENT}║         FILE MANAGER                   ║${RESET}"
    echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}"
    echo -e "${THEME_SECONDARY}Current: $(pwd)${RESET}\n"
    
    ls -lh --color=auto 2>/dev/null || ls -lh
    
    echo -e "\n${THEME_PRIMARY}[cd]${RESET} Change dir  ${THEME_PRIMARY}[mkdir]${RESET} New dir  ${THEME_PRIMARY}[rm]${RESET} Delete  ${THEME_PRIMARY}[0]${RESET} Back"
    printf "${THEME_SECONDARY}Command>${RESET} "
    read -r fm_cmd fm_arg
    
    case $fm_cmd in
      cd) cd "$fm_arg" 2>/dev/null || echo -e "${RED}Directory not found${RESET}" ;;
      mkdir) mkdir -p "$fm_arg" && echo -e "${THEME_PRIMARY}Created: $fm_arg${RESET}" ;;
      rm) rm -rf "$fm_arg" && echo -e "${THEME_PRIMARY}Deleted: $fm_arg${RESET}" ;;
      0) break ;;
    esac
    sleep 1
  done
}

# =========== LIVE MONITOR ============
live_monitor() {
  clear
  echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
  echo -e "${THEME_ACCENT}║      LIVE SYSTEM MONITOR               ║${RESET}"
  echo -e "${THEME_ACCENT}║      Press Ctrl+C to exit              ║${RESET}"
  echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
  
  while true; do
    tput cup 4 0
    ram_used=$(free -m 2>/dev/null | awk '/^Mem:/ {print $3}')
    ram_total=$(free -m 2>/dev/null | awk '/^Mem:/ {print $2}')
    cpu=$(top -bn1 2>/dev/null | grep "Cpu(s)" | awk '{print $2 + $4}' || echo "0")
    
    echo -e "${THEME_PRIMARY}RAM: ${THEME_SECONDARY}$ram_used${RESET}/${THEME_PRIMARY}$ram_total MB${RESET}  "
    echo -e "${THEME_PRIMARY}CPU: ${THEME_SECONDARY}${cpu%.*}%${RESET}           "
    echo -e "${THEME_SECONDARY}$(date '+%H:%M:%S')${RESET}           "
    
    sleep 1
  done
}

# =========== NOTES MANAGER ============
notes_manager() {
  while true; do
    clear
    echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
    echo -e "${THEME_ACCENT}║         NOTES / TODO                   ║${RESET}"
    echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
    
    if [[ -f "$NOTES_FILE" ]]; then
      cat -n "$NOTES_FILE"
    else
      echo -e "${THEME_SECONDARY}No notes yet.${RESET}"
    fi
    
    echo -e "\n${THEME_PRIMARY}[a]${RESET} Add  ${THEME_PRIMARY}[d]${RESET} Delete line  ${THEME_PRIMARY}[c]${RESET} Clear all  ${THEME_PRIMARY}[0]${RESET} Back"
    printf "${THEME_SECONDARY}Choice>${RESET} "
    read -r note_choice
    
    case $note_choice in
      a)
        printf "${THEME_SECONDARY}Enter note:${RESET} "
        read -r note_text
        echo "$note_text" >> "$NOTES_FILE"
        ;;
      d)
        printf "${THEME_SECONDARY}Line number to delete:${RESET} "
        read -r line_num
        sed -i "${line_num}d" "$NOTES_FILE" 2>/dev/null
        ;;
      c)
        > "$NOTES_FILE"
        echo -e "${THEME_PRIMARY}All notes cleared!${RESET}"
        sleep 1
        ;;
      0) break ;;
    esac
  done
}

# =========== SHORTCUTS MANAGER ============
shortcuts_manager() {
  while true; do
    clear
    echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
    echo -e "${THEME_ACCENT}║      COMMAND SHORTCUTS                 ║${RESET}"
    echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
    
    if [[ -f "$SHORTCUTS_FILE" ]]; then
      cat -n "$SHORTCUTS_FILE"
    else
      echo -e "${THEME_SECONDARY}No shortcuts yet.${RESET}"
    fi
    
    echo -e "\n${THEME_PRIMARY}[a]${RESET} Add  ${THEME_PRIMARY}[r]${RESET} Run  ${THEME_PRIMARY}[d]${RESET} Delete  ${THEME_PRIMARY}[0]${RESET} Back"
    printf "${THEME_SECONDARY}Choice>${RESET} "
    read -r sc_choice
    
    case $sc_choice in
      a)
        printf "${THEME_SECONDARY}Enter command:${RESET} "
        read -r cmd_text
        echo "$cmd_text" >> "$SHORTCUTS_FILE"
        ;;
      r)
        printf "${THEME_SECONDARY}Line number to run:${RESET} "
        read -r line_num
        cmd=$(sed -n "${line_num}p" "$SHORTCUTS_FILE")
        echo -e "${THEME_PRIMARY}Running: $cmd${RESET}"
        eval "$cmd"
        echo -e "\n${THEME_ACCENT}Press Enter...${RESET}"
        read
        ;;
      d)
        printf "${THEME_SECONDARY}Line number to delete:${RESET} "
        read -r line_num
        sed -i "${line_num}d" "$SHORTCUTS_FILE" 2>/dev/null
        ;;
      0) break ;;
    esac
  done
}

# =========== SSH MANAGER ============
ssh_manager() {
  while true; do
    clear
    echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
    echo -e "${THEME_ACCENT}║         SSH MANAGER                    ║${RESET}"
    echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
    
    if [[ -f "$SSH_FILE" ]]; then
      cat -n "$SSH_FILE"
    else
      echo -e "${THEME_SECONDARY}No SSH hosts saved.${RESET}"
    fi
    
    echo -e "\n${THEME_PRIMARY}[a]${RESET} Add  ${THEME_PRIMARY}[c]${RESET} Connect  ${THEME_PRIMARY}[d]${RESET} Delete  ${THEME_PRIMARY}[0]${RESET} Back"
    printf "${THEME_SECONDARY}Choice>${RESET} "
    read -r ssh_choice
    
    case $ssh_choice in
      a)
        printf "${THEME_SECONDARY}Enter SSH command (e.g. ssh user@host):${RESET} "
        read -r ssh_cmd
        echo "$ssh_cmd" >> "$SSH_FILE"
        ;;
      c)
        printf "${THEME_SECONDARY}Line number to connect:${RESET} "
        read -r line_num
        ssh_cmd=$(sed -n "${line_num}p" "$SSH_FILE")
        echo -e "${THEME_PRIMARY}Connecting: $ssh_cmd${RESET}"
        eval "$ssh_cmd"
        ;;
      d)
        printf "${THEME_SECONDARY}Line number to delete:${RESET} "
        read -r line_num
        sed -i "${line_num}d" "$SSH_FILE" 2>/dev/null
        ;;
      0) break ;;
    esac
  done
}

# =========== DOWNLOAD MANAGER ============
download_manager() {
  clear
  echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
  echo -e "${THEME_ACCENT}║       DOWNLOAD MANAGER                 ║${RESET}"
  echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
  
  printf "${THEME_SECONDARY}Enter URL to download:${RESET} "
  read -r dl_url
  
  if command -v aria2c >/dev/null 2>&1; then
    aria2c -x 16 -s 16 "$dl_url"
  elif command -v wget >/dev/null 2>&1; then
    wget "$dl_url"
  else
    echo -e "${RED}No download tool found. Install: pkg install wget${RESET}"
  fi
  
  echo -e "\n${THEME_ACCENT}Press Enter...${RESET}"
  read
}

# =========== SECURITY MENU ============
security_menu() {
  while true; do
    clear
    echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
    echo -e "${THEME_ACCENT}║         SECURITY TOOLS                 ║${RESET}"
    echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
    
    echo -e "${THEME_PRIMARY}[1]${RESET} Set/Change Password"
    echo -e "${THEME_PRIMARY}[2]${RESET} Remove Password"
    echo -e "${THEME_PRIMARY}[3]${RESET} Generate Strong Password"
    echo -e "${THEME_PRIMARY}[4]${RESET} Encrypt File (gpg)"
    echo -e "${THEME_PRIMARY}[5]${RESET} Clear Command History"
    echo -e "${THEME_PRIMARY}[0]${RESET} Back\n"
    
    printf "${THEME_SECONDARY}Choice>${RESET} "
    read -r sec_choice
    
    case $sec_choice in
      1)
        printf "\n${THEME_SECONDARY}Enter new password:${RESET} "
        read -s new_pass
        echo "$new_pass" > "$PASSWORD_FILE"
        chmod 600 "$PASSWORD_FILE"
        echo -e "\n${THEME_PRIMARY}Password set!${RESET}"
        sleep 2
        ;;
      2)
        rm -f "$PASSWORD_FILE"
        echo -e "${THEME_PRIMARY}Password removed!${RESET}"
        sleep 2
        ;;
      3)
        pass=$(tr -dc 'A-Za-z0-9!@#$%^&*' < /dev/urandom | head -c 16)
        echo -e "\n${THEME_PRIMARY}Generated: ${THEME_ACCENT}$pass${RESET}"
        echo -e "${THEME_ACCENT}Press Enter...${RESET}"
        read
        ;;
      4)
        printf "\n${THEME_SECONDARY}Enter filename to encrypt:${RESET} "
        read -r file_enc
        if command -v gpg >/dev/null 2>&1; then
          gpg -c "$file_enc"
          echo -e "${THEME_PRIMARY}File encrypted!${RESET}"
        else
          echo -e "${RED}gpg not installed${RESET}"
        fi
        sleep 2
        ;;
      5)
        > ~/.bash_history
        history -c
        echo -e "${THEME_PRIMARY}History cleared!${RESET}"
        sleep 2
        ;;
      0) break ;;
    esac
  done
}

# =========== BACKUP SYSTEM ============
backup_system() {
  clear
  echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
  echo -e "${THEME_ACCENT}║         BACKUP SYSTEM                  ║${RESET}"
  echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
  
  backup_dir="$HOME/x1_backup_$(date +%Y%m%d_%H%M%S)"
  mkdir -p "$backup_dir"
  
  echo -e "${THEME_SECONDARY}Backing up configuration files...${RESET}"
  
  cp -r "$CONFIG_DIR" "$backup_dir/" 2>/dev/null
  cp ~/.bashrc "$backup_dir/" 2>/dev/null
  cp ~/.bash_profile "$backup_dir/" 2>/dev/null
  
  echo -e "${THEME_PRIMARY}Backup saved to: $backup_dir${RESET}"
  echo -e "${THEME_ACCENT}Press Enter...${RESET}"
  read
}

# =========== SPEEDTEST ============
speedtest_run() {
  clear
  echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
  echo -e "${THEME_ACCENT}║         SPEED TEST                     ║${RESET}"
  echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
  
  if command -v speedtest-cli >/dev/null 2>&1; then
    speedtest-cli
  else
    echo -e "${RED}speedtest-cli not installed${RESET}"
    echo -e "${THEME_SECONDARY}Install with: pkg install speedtest-cli${RESET}"
  fi
  
  echo -e "\n${THEME_ACCENT}Press Enter...${RESET}"
  read
}

# =========== THEME SELECTOR ============
theme_selector() {
  while true; do
    clear
    echo -e "${THEME_ACCENT}╔════════════════════════════════════════╗${RESET}"
    echo -e "${THEME_ACCENT}║         THEME SELECTOR                 ║${RESET}"
    echo -e "${THEME_ACCENT}╚════════════════════════════════════════╝${RESET}\n"
    
    echo -e "${GREEN}[1]${RESET} Green (Matrix)"
    echo -e "${RED}[2]${RESET} Red (Alert)"
    echo -e "${BLUE}[3]${RESET} Blue (Ocean)"
    echo -e "${MAGENTA}[4]${RESET} Purple (Cyber)"
    echo -e "${CYAN}[5]${RESET} Cyan (Ice)"
    echo -e "${YELLOW}[0]${RESET} Back\n"
    
    printf "${THEME_SECONDARY}Choice>${RESET} "
    read -r theme_choice
    
    case $theme_choice in
      1) THEME_PRIMARY=$GREEN; THEME_SECONDARY=$CYAN; THEME_ACCENT=$YELLOW ;;
      2) THEME_PRIMARY=$RED; THEME_SECONDARY=$YELLOW; THEME_ACCENT=$RED ;;
      3) THEME_PRIMARY=$BLUE; THEME_SECONDARY=$CYAN; THEME_ACCENT=$BLUE ;;
      4) THEME_PRIMARY=$MAGENTA; THEME_SECONDARY=$CYAN; THEME_ACCENT=$MAGENTA ;;
      5) THEME_PRIMARY=$CYAN; THEME_SECONDARY=$BLUE; THEME_ACCENT=$CYAN ;;
      0) save_theme; break ;;
    esac
    
    if [[ $theme_choice != 0 ]]; then
      save_theme
      echo -e "${THEME_PRIMARY}Theme applied!${RESET}"
      sleep 1
      break
    fi
  done
}

# =========== EXIT PROGRAM ============
exit_program() {
  clear
  echo -e "${RED}Disconnecting...${RESET}"
  sleep 0.5
  echo -e "${CYAN}Session Terminated.${RESET}"
  exit 0
}

# ============= START ============
check_password
main_menu
