# Создание Wireguard Server
#!/bin/bash

Server() {
    echo "Вы первый раз запускаете скрипт?"
    read first_answer
    if [[ $first_answer == "Да" || $first_answer == "дА" || $first_answer == "да" || $first_answer == "ДА" || $first_answer == "Yes" || $first_answer == "Y" || $first_answer == "yes" || $first_answer == "y" ]]; then
        echo "Создадим сервер с протоколом VPN WireGuard"
    else
        "Pi"
    fi
}
Server   

        #Узнаем какую операционку мы используем и после этого обновим репозиторий и пакеты
        function OS {
            os_info=$(lsb_release -a)
            if [[ $os_info == *"CentOS"* || $os_info == *"Fedora"* ]]; then
                yum update
            elif
                [[ $os_info == *"Debian"* || $os_info == *"Ubuntu"* ]]; then
                    sudo apt update && sudo apt upgrade -y
            else
                echo "Вы пользуетесь MacOS"
            fi
        }
        OS







        #Раскоментируем строчку, чтобы пропускать трафик
        function IP_Forward {

            forward=$(sudo sed -i '/^#net.ipv4.ip_forward=1/c\net.ipv4.ip_forward=1' /etc/sysctl.conf)
            sysctl_role=$(sudo sysctl -p)

            echo "$forward"
            echo "$sysctl_role"

        }
        IP_Forward




        echo "Изменим разрешение на директорию"
        sudo chmod a+rwx /etc/wireguard
        cd /etc/wireguard
        umask 077
        echo ""
        echo "Генерирую ключи"
        wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey



        #Скачаем пакеты относящиеся к wireguard на нашем сервере
        function WireGuard {
            os_info=$(lsb_release -a)
            if [[ $os_info == *"CentOS"* || $os_info == *"Fedora"* ]]; then
                yum install wireguard wireguard-tools qrencode
            elif
                [[ $os_info == *"Debian"* || $os_info == *"Ubuntu"* ]]; then
                    sudo apt install wireguard wireguard-tools qrencode
            else
                echo "Вы используете MacOS"
            fi
        }
        WireGuard

        echo ""

        function IP_address {
            echo -e "Какой IP адрес хочешь использовать?\n1) 10.0.0.0/8 - от 10.0.0.0 до 10.255.255.255\n2) 172.16.0.0/12 - от 172.16.0.0 до 172.31.255.255\n3) 192.168.0.0/16 - от 192.168.0.0 до 192.168.255.255\n4) Стандартная сетка - 192.168.0.0/24 - от 192.168.0.0 до 192.168.0.255"
            read ip_address
            if [[ $ip_address =~ ^10\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})$ || $ip_address =~ ^172\.([1-3][0-9]?)\.([0-9]{1,3})\.([0-9]{1,3})$ || $ip_address =~ ^192\.168\.([0-9]{1,3})\.([0-9]{1,3})$ ]]; then
                echo "Ваш IP-адрес: $ip_address"
            else
                echo "Попробуйте еще раз"
                IP_address  # рекурсивный вызов функции
            fi
        }
        IP_address

        echo ""
                DNS_Server1() {
            PS3="Какой Первичный DNS сервер предпочитаете? :"
            local selection
            select selection in Google CloudFlare Quad9 OpenDNS AdGuard 
            do
                case $selection in
                Google)
                    DNS1="8.8.8.8"
                        echo "Вы выбрали Google DNS - $DNS1"
                        break
                        ;; 
                CloudFlare)
                    DNS1="1.1.1.1"
                        echo "Вы выбрали CloudFlare DNS - $DNS1"
                        break
                        ;;
                Quad9)
                    DNS1="9.9.9.9"
                        echo "Вы выбрали Quad9 DNS - $DNS1"
                        break
                        ;;
                OpenDNS)
                    DNS1="208.67.222.222"
                        echo "Вы выбрали OpenDNS - $DNS1"
                        break
                        ;;
                AdGuard)
                    DNS1="176.103.130.130"
                        echo "Вы выбрали AdGuard DNS - $DNS1"
                        break
                        ;;
                *)
                        echo "Недопустимый выбор"
                        ;;
                esac
            done
            echo "Выбранный Первичный DNS сервер: $DNS1"
        }
        DNS_Server1









        DNS_Server2() {
            PS3="Какой Вторичный DNS сервер предпочитаете? :"
            local selection
            select selection in Google CloudFlare Quad9 OpenDNS AdGuard
            do
                case $selection in
                Google)
                    DNS2="8.8.4.4"
                        echo "Вы выбрали Google DNS - $DNS2"
                        break
                        ;; 
                CloudFlare)
                    DNS2="1.0.0.1"
                        echo "Вы выбрали CloudFlare DNS - $DNS2"
                        break
                        ;;
                Quad9)
                    DNS2="149.112.112.112"
                        echo "Вы выбрали Quad9 DNS - $DNS2"
                        break
                        ;;
                OpenDNS)
                    DNS2="208.67.220.220"
                        echo "Вы выбрали OpenDNS - $DNS2"
                        break
                        ;;
                AdGuard)
                    DNS2="176.103.130.131"
                        echo "Вы выбрали AdGuard DNS - $DNS2"
                        break
                        ;;
                *)
                        echo "Недопустимый выбор"
                        ;;
                esac
            done
            echo "Выбранный Вторичный DNS сервер: $DNS2"
        }
        DNS_Server2

        Port() {
            PS3="Какой порт предпочитаете использовать? :"
            select per1 in Стандартный Кастомный; do
                case $per1 in
                    Стандартный)
                        per2="51820"
                        ;;
                    Кастомный)
                        echo "Введите номер порта:"
                        read per2
                        if ! [[ $per2 =~ ^[0-9]+$ ]] || (( per2 < 1 || per2 > 65535 )); then
                            echo "Недопустимый порт. Попробуйте еще раз."
                            continue
                        fi
                        ;;
                    *)
                        echo "Недопустимый выбор"
                        continue
                        ;;
                esac
                echo "Вы выбрали порт: $per2"
                break
            done
        }
        Port

        function directory {
            echo "В какой директории будем держать серверный конфиг?"
            read dir
            directory_file="$dir"
                if [[ $directory_file == /* ]]; then
                            mkdir -p "$directory_file"
                        else
                            echo "Конфиг запишется в /etc/wireguard"
                    directory_file="/etc/wireguard"
                    mkdir -p "$directory_file"
                        fi
        }
        directory

#function New_Clients {
#echo "Создадим отдельную папку для девайса. Напишите полный маршрут, где будут храниться ключи и конфигурационный файл, иначе по дефолту маршрут будет - /etc/wireguard/Сlients: "
#read directory1
    #papka_clients="$directory1"
        #if [[ $papka_clients == /* ]]; then
            #mkdir -p "$papka_clients"
        #else
            #echo "Папка запишется в /etc/wireguard/Сlients"
            #papka_clients="/etc/wireguard/Clients"
            #mkdir -p "$papka_clients"
        #fi
#}
#New_Clients

        function Maska {
            echo "Какую маску вы хотите выбрать? Напишите свой выбор. От 1 - до 32"
            read maska_podseti
            if [[ maska_podseti -lt 1 || maska_podseti -gt 32 ]] ; then
                echo "Вы выбрали не правильную маску! Повторите свой выбор"
                Maska
                
            else
                net_mask="$maska_podseti"
                echo "Маска подсети выбрана!"
            fi
        }
        Maska

        function Config-Server  {
            echo "Создадим серверный конфиг, как назовем его?"
            read config
            configuration_file="${config}.conf"
            echo ""
            echo "В директории $directory_file будtт конфигурационный файл"
            conf_file="${directory_file}/${configuration_file}"
            touch "$conf_file"
            echo "[Interface]" >> "$conf_file"
            echo "Address = $ip_address/$maska_podseti" >> "$conf_file"
            echo "SaveConfig = true" >> "$conf_file"
            echo "ListenPort = $per2" >> "$conf_file"
            priv_key=$(sudo cat /etc/wireguard/privatekey)
            echo "PrivateKey = $priv_key" >> "$conf_file"
            interface=$(ip -o -4 route show to default | awk '{print $5}')
            PostUP="PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o $interface -j MASQUERADE"
            PostDown="PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o $interface -j MASQUERADE"
            echo "$PostUP" >> "$conf_file"
            echo "$PostDown" >> "$conf_file"
            sudo chmod 600 /etc/wireguard/privatekey
            sudo chmod 600 "${directory_file}"/"${configuration_file}"
        }
        Config-Server

###############

        
        
















        IP_address_Client() {
        echo -e 'Какой IP адрес хочешь задать клиенту? Ваш ранее выбранный IP_Address для сервера - $ip_address\n1) 10.0.0.1/8 - от 10.0.0.2 до 10.255.255.255\n2) 172.16.0.1/12 - от 172.16.0.2 до 172.31.255.255\n3) 192.168.0.1/16 - от 192.168.0.2 до 192.168.255.255\n4) Стандартная сетка - 192.168.0.1/24 - от 192.168.0.2 до 192.168.0.255'
        read ip_address_clients
        if [[ $ip_address_clients =~ ^10\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([1-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])$ || ip_address_clients =~ ^172\.(1[6-9]|2[0-9]|3[0-1])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([1-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])$ || ip_address_clients =~ ^192\.(1[6-9][8-9])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([1-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])$ ]]; then
        #^10\.: Это начало регулярного выражения, которое проверяет, что IP-адрес начинается с "10."
        #([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5]): Это группа, которая проверяет, что IP-адрес состоит из одного или двух чисел от 0 до 9, или двух чисел от 10 до 99, или трех чисел от 100 до 249, или четырех чисел от 250 до 255
        #\.: Это символ точки, который отделяет октеты IP-адреса
        #$: Это конец регулярного выражения, которое проверяет, что IP-адрес заканчивается тремя группами чисел от 0 до 255, разделенных точками
        #Для проверки того, что IP-адрес начинается с определенной цифры. Вы можете сделать это с помощью оператора =~
        #Здесь ^ обозначает начало строки, а . - любой символ. Таким образом, этот код проверяет, начинается ли IP-адрес с 10, 172 или 192
            echo "Ваш IP-адресс: $ip_address_clients"
            echo ""
        else
            echo ""
            echo "Попробуйте еще раз"
            echo ""
        fi
        }
        IP_address_Client                    


                                                        function New_Clients {
                                                        echo "Создадим отдельную папку для девайса. Напишите полный маршрут, где будут храниться ключи и конфигурационный файл, иначе по дефолту маршрут будет - /etc/wireguard/Сlients: "
                                                        read directory1
                                                        papka_clients="$directory1"
                                                            if [[ $papka_clients == /* ]]; then
                                                                mkdir -p "$papka_clients"
                                                            else
                                                                echo "Папка запишется в /etc/wireguard/Сlients"
                                                                papka_clients="/etc/wireguard/Clients"
                                                                mkdir -p "$papka_clients"
                                                            fi
                                                        }
                                                        New_Clients

                                                        function WG_Key {
                                                            echo "Как назовем публичный и приватный ключи клиента?"
                                                            read key
                                                            key_client="$key"
                                                            wg genkey | tee $papka_clients/${key_client}.key | wg pubkey | tee $papka_clients/${key_client}.key.pub
                                                        }
                                                        WG_Key








                                                        function Config_Client {
                                                            cpub=$(sudo cat /etc/wireguard/publickey)
                                                            #cvip=$(sudo cat /etc/wireguard/privatekey)
                                                            cvipclinet=$(sudo cat $papka_clients/${key_client}.key)
                                                            #ip_host=$(hostname -I)
                                                            
                                                            #Если у вашего сервера несколько IP-адресов, вам нужно выбрать нужный адрес. Например, если вы хотите использовать первый IP-адрес из списка, используйте:
                                                            ip_host=$(hostname -I | awk '{print $1}')

                                                            setter=$(wg set $config peer $papka_clients/${key_client}.key.pub allowed-ips $ip_address_clients)
                                                            echo "Создадим клиентский конфиг, как назовем его?"
                                                            read config_client
                                                            confclient_file="${config_client}.conf"
                                                            echo ""
                                                            config_clients="$papka_clients/$confclient_file"
                                                            touch "$config_clients"

                                                            
                                                            
                                                            echo "[Interface]" >> "$config_clients"
                                                            #echo "PrivateKey = $papka_clients/${key_client}.key" >> "$config_clients"
                                                            echo "PrivateKey = $cvipclinet" >> "$config_clients"
                                                            echo "Address = $ip_address_clients/$maska_podseti" >> "$config_clients"
                                                            echo "DNS = $DNS1, $DNS2" >> "$config_clients"
                                                            echo "[Peer]" >> "$config_clients"
                                                            echo "PublicKey = $cpub" >> "$config_clients"
                                                            echo "AllowedIPs = 0.0.0.0/0" >> "$config_clients"
                                                            echo "Endpoint = $ip_host:$per2" >> "$config_clients"
                                                            echo "$setter"
                                                        }
                                                        Config_Client


                                                            function WG-QUICK {
                                                                    wgquickdown=$(wg-quick down "$config")
                                                                    wgquickup=$(wg-quick up "$config")
                                                                    start=$(sudo systemctl start wg-quick@"$config")
                                                                    stop=$(sudo systemctl stop wg-quick@"$config")
                                                                    enable=$(sudo systemctl enable wg-quick@"$config")
                                                                        echo "$wgquickdown"
                                                                        if [[ $wgquickdown == *"does not exist"* || $wgquickdown == *"not a Wireguard interface"* ]]; then
                                                                            echo "$enable"
                                                                            echo "$start"
                                                            #                echo "$wgquickup"
                                                                        else
                                                                            echo "$wgquickdown"
                                                                            echo "$enable"
                                                                            echo "$start"
                                                                        fi
                                                                    }
                                                                    WG-QUICK                                                        




                                                        function QrenCode {
                                                            echo "Хотите открыть QR код для подключения?"
                                                            read qr
                                                            qr_code="qrencode -t ansiutf8 < \"$papka_clients"/"$confclient_file\""
                                                            if [[ $qr == "Да" || $qr == "дА" || $qr == "да" || $qr == "ДА" || $qr == "Yes" || $qr == "Y" || $qr == "yes" || $qr == "y" ]]; then
                                                                eval "$qr_code"
                                                            else
                                                                echo "Test"
                                                            fi
                                                        }
                                                        QrenCode
