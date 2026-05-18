## 08. Polish

sudo timedatectl set-timezone Europe/Paris

cat >> ~/.bashrc << 'EOF'
export HISTCONTROL=ignoreboth
export HISTSIZE=10000
export HISTTIMEFORMAT="%F %T  "
EOF
source ~/.bashrc