# 🌿 Horta Inteligente - ESP32 RainMaker (Sistema Failsafe)

![ESP32](https://img.shields.io/badge/ESP32-C3-blue) ![RainMaker](https://img.shields.io/badge/ESP-RainMaker-brightgreen) ![Status](https://img.shields.io/badge/Status-Est%C3%A1vel-success)

Sistema de irrigação automatizado de alta resiliência utilizando **ESP32-C3 SuperMini** e a plataforma **ESP RainMaker**. O projeto é dividido em dois módulos (Sensor e Relé) que se comunicam via Nuvem (App) e via Rede Local (UDP Broadcast), garantindo que a sua horta continue sendo regada (e protegida contra inundações) mesmo se a sua conexão de internet cair.

## ✨ Principais Funcionalidades

* 📱 **Controle via Nuvem (App):** Monitoramento e controle total de qualquer lugar do mundo pelo app ESP RainMaker.
* 🛡️ **Comunicação Híbrida (Nuvem + Local):** O Sensor envia os dados para a nuvem, mas também "grita" comandos na rede Wi-Fi local (UDP Broadcast). O Relé atua instantaneamente ouvindo a rede local, sem depender da latência da internet.
* 🚨 **Failsafe & Watchdog (Cão de Guarda):** Se o Relé parar de receber o sinal de "vida" (Ping UDP) do Sensor por mais de 30 segundos (ex: sensor queimou, placa travou, roteador desligou), ele **corta a água automaticamente**, evitando inundações.
* 🧠 **Cura de Amnésia (Memória NVS):** Todas as configurações (tempos, faixas de sensores, canais) são salvas em memória Flash. Após uma queda de energia, o sistema volta exatamente como estava configurado, sem precisar de sincronização com a nuvem.
* 🎛️ **Zonamento Dinâmico (Walkie-Talkie):** Suporta até 10 zonas diferentes. O pareamento entre Sensor e Relé é feito diretamente por um Slider no aplicativo (Canais de 1 a 10), sem precisar alterar o código-fonte!
* ⏳ **Ciclos Inteligentes:** Irrigação por temperatura conta com ciclos de **Regar** e **Pausar** (ex: rega 5 min, pausa 15 min), evitando o encharcamento do solo em dias muito quentes.

## 🏗️ Arquitetura do Sistema

O projeto é composto por dois códigos distintos, rodando de forma independente, mas sincronizada:

### 1. ESP-SENSOR (O Cérebro / Megafone)
Fica na horta. Lê a umidade do solo e a temperatura/umidade do ar (DHT22). 
Processa as regras definidas no app (ex: se solo < 30% ou Temp > 32°C -> Ligar água). 
Além de atualizar o RainMaker, ele grita na rede local a cada 2 segundos (ex: `ZONA:1:ON`).

### 2. ESP-RELÉ (O Músculo / Ouvinte)
Fica próximo à bomba de água ou válvula solenoide. 
Não precisa processar regras complexas, apenas escuta o canal configurado (ex: Zona 1). Se o comando for `ON`, ele aciona o NPN e liga o relé. Se a internet cair, ele continua operando 100% via rede local. 

## 🛠️ Hardware Necessário

* 2x Placas **ESP32-C3 SuperMini**
* 1x Sensor de Umidade do Solo (Capacitivo recomendado)
* 1x Sensor de Temperatura/Umidade **DHT22**
* 1x Módulo Relé (acionamento em nível baixo/Active LOW)
* 1x Transistor NPN (para acionar o Relé via 3.3V do ESP32 de forma segura) + Resistor 4.7k
* Fios, protoboard e fontes de alimentação (5V/3.3V).

## 🚀 Como Configurar e Usar

1. **Gravação:** Grave o firmware `ESP-SENSOR` na primeira placa e o `ESP-RELE` na segunda placa usando a Arduino IDE.
2. **Provisionamento:** Abra o app ESP RainMaker no seu smartphone, clique em `+` (Add Device), pareie ambas as placas via Bluetooth e insira as credenciais do seu Wi-Fi.
3. **Pareamento das Placas:**
   * No app, abra o dispositivo "Sensor Horta" e mude o "Canal de Comunicação" para `1`.
   * Abra o dispositivo do Relé e mude o "Canal de Comunicação" também para `1`.
   * *Pronto! A partir desse momento, as duas placas estão pareadas e conversando em rede local.*
4. **Regras Automáticas:** Defina no Sensor se quer trabalhar no Modo Automático ou Manual. Ajuste os limites de umidade do solo e temperatura para acionar a sua bomba no tempo correto.

## ⚙️ Controles Físicos (Botão BOOT na Placa)

Ambas as placas possuem um sistema de recuperação nativo de emergência através do botão físico `BOOT` (GPIO 9):
* **3 Cliques curtos:** Ativa o modo temporário **SoftAP** (para parear o Wi-Fi sem Bluetooth, caso precise trocar de rede e o QR code tenha sido perdido).
* **Segurar por 3 a 4 segundos:** **Reset de Wi-Fi**. Apaga a rede atual e entra em modo de provisionamento novamente. Mantém intactas as regras de irrigação.
* **5 a 7 Cliques curtos:** **Factory Reset Total**. Formata o ESP32 de fábrica. Apaga o pareamento do RainMaker, destrói o banco de dados interno e volta o sistema para o estado de segurança absoluto (Tudo OFF).

## ⚠️ Detalhes de Segurança e Engenharia
* Todos os parâmetros sensíveis contam com limite máximo e mínimo travados no firmware (Impossível o usuário bugar o sistema configurando a temperatura mínima maior que a máxima pelo App).
* **O Princípio do Fail-Safe:** Se o módulo do relé for reiniciado por queda de energia ou travamento elétrico, ele forçosamente iniciará com a bomba de água desligada (`OFF`), ligando apenas após reestabelecer comunicação auditiva de segurança com o Sensor ou receber ordem direta da Nuvem.

---
*Projeto Open Source desenvolvido para unir a conveniência da IoT baseada em nuvem com a robustez e resiliência da comunicação de rede local industrial.*
