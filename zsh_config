**Configurando o Zsh no Arch Linux para Ficar Igual ao Manjaro**

Se você quer configurar o **Zsh** para ficar no estilo do **Manjaro**, você pode seguir alguns passos. O Manjaro usa o Zsh como shell padrão e tem uma configuração personalizada que inclui o **Oh My Zsh** e um prompt estilizado.

---

## **1. Instalar o Zsh e Definir como Shell Padrão**
Se o Zsh ainda não estiver instalado, execute:
```bash
sudo pacman -S zsh git
```

Defina o Zsh como shell padrão:
```bash
chsh -s /bin/zsh
```
Depois, reinicie o terminal ou rode `zsh` para iniciar o shell.

---

## **2. Instalar o Oh My Zsh**
O Manjaro usa o **Oh My Zsh** para gerenciar temas e plugins. Para instalar:
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

---

## **3. Configurar o Tema do Manjaro**
O Manjaro usa o tema **powerlevel10k**, então instale-o:
```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

Agora, edite o `~/.zshrc` e altere a linha do tema:
```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Aplique as mudanças:
```bash
source ~/.zshrc
```

Na próxima vez que abrir o terminal, o **Powerlevel10k** iniciará um assistente de configuração. Basta seguir as opções para personalizar o visual como no Manjaro. Para reconfigurar depois:
```bash
p10k configure
```

---

## **4. Ativar Plugins do Manjaro**
O Manjaro usa alguns plugins no Zsh. Edite o `~/.zshrc` e altere a linha de plugins:
```bash
plugins=(git archlinux colored-man-pages zsh-autosuggestions zsh-syntax-highlighting)
```

Agora, instale os plugins manualmente:
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

Recarregue o shell:
```bash
source ~/.zshrc
```

---

## **5. Corrigir Delete, Home e End no GNOME Terminal**
Se as teclas `Delete`, `Home` e `End` não funcionarem corretamente, adicione isto ao final do `~/.zshrc`:
```bash
# Corrigir Delete, Home e End no GNOME Terminal
bindkey "^[[3~" delete-char
bindkey "^[[H" beginning-of-line
bindkey "^[[F" end-of-line
bindkey "^[[1~" beginning-of-line  
bindkey "^[[4~" end-of-line  
bindkey "^[[7~" beginning-of-line  
bindkey "^[[8~" end-of-line  
```

Depois, recarregue o shell:
```bash
source ~/.zshrc
```

---

## **6. Adicionar a Configuração do Manjaro**
O Manjaro tem um pacote chamado **zsh-config** que define algumas configurações extras. Você pode baixar o `.zshrc` deles e usá-lo no Arch:
```bash
wget -O ~/.zshrc https://gitlab.manjaro.org/packages/community/zsh-config/-/raw/master/zshrc
source ~/.zshrc
```

Se quiser adicionar mais personalizações, basta editar o arquivo com:
```bash
nano ~/.zshrc
```

---

## **Conclusão**
Seguindo esses passos, seu **Zsh** no Arch Linux ficará idêntico ao do Manjaro, incluindo o **Oh My Zsh**, o **Powerlevel10k**, plugins e atalhos corrigidos para o **GNOME Terminal**. Agora seu terminal estará estilizado e funcional como no Manjaro!

