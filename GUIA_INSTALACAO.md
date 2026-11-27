# Guia de instalação — Samsung TV Remote (TecnoSimples)

Driver Hubitat para TVs **Samsung Tizen (2016 em diante)**: liga, desliga e controla a TV pela rede local.

São três etapas, nesta ordem:

1. [Preparar a TV](#1-preparar-a-tv) — 5 min
2. Instalar o driver: [pelo HPM](#2a-instalar-pelo-hpm-recomendado) (recomendado) **ou** [sem HPM](#2b-instalar-sem-hpm)
3. [Criar e configurar o device](#3-criar-e-configurar-o-device) — 3 min

---

## 1. Preparar a TV

### 1.1 Ativar "Ligar com Celular"

Sem isso, a TV **não liga pela rede** depois que entra em repouso profundo (cerca de 20 s após desligar).

No controle remoto da TV:

1. **Configurações** (ícone de engrenagem) → **Todas as configurações**
2. **Geral** → **Rede** → **Configurações Avançadas** (em alguns modelos: *Configurações de Especialista*)
3. Ative **Ligar com Celular** (*Power On with Mobile*)

> Os nomes dos menus mudam um pouco conforme o ano da TV. Se não achar, procure por "Ligar com Celular"
> ou "Power On with Mobile" na busca das Configurações.

### 1.2 Deixar o IP da TV fixo

O driver encontra a TV pelo IP. Se o IP mudar, a TV deixa de responder.

- **Recomendado:** no roteador, crie uma **reserva de DHCP** para a TV (associar o MAC da TV a um IP).
- Para ver o IP atual da TV: **Configurações → Geral → Rede → Status da Rede → Configurações de IP**.

### 1.3 Ligar a TV por cabo de rede (opcional, mas melhor)

Pelo cabo, o Wake-on-LAN é mais confiável do que pelo Wi-Fi. Se a TV estiver no Wi-Fi e às vezes não
ligar, este é o primeiro ponto a verificar.

---

## 2a. Instalar pelo HPM (recomendado)

Com o HPM (Hubitat Package Manager), as atualizações do driver chegam pelo próprio HPM.

1. No Hubitat: **Apps** → **Hubitat Package Manager**
2. **Install** → **From a URL**
3. Cole a URL abaixo e clique em **Next**:
   ```
   https://raw.githubusercontent.com/tecnosimples/hubitat_samsung_tv/main/packageManifest.json
   ```
4. Confirme a instalação e aguarde a mensagem de conclusão.

Siga para a [etapa 3](#3-criar-e-configurar-o-device).

### Já usa o driver original do Dave Gutheinz?

Para manter os devices existentes (com regras e dashboards):

1. HPM → **Package Manager Settings** → **Remove a Matched Package** → selecione **Samsung TV Remote**
   (o pacote antigo) → Next. Isso só desfaz o vínculo no HPM; não apaga nada.
2. Instale este pacote pelos passos acima.
3. Em cada device da TV: **Device Info** → **Type** → `Samsung TV Remote (TecnoSimples)` → **Save Device**.
4. Com a TV **ligada**, clique em **Save Preferences** no device.

> Sem o passo 1, um "Update" do HPM pode trocar este driver de volta pelo original.

---

## 2b. Instalar sem HPM

Nesse modo, as atualizações **não chegam sozinhas**. Para atualizar, repita a importação.

1. No Hubitat: **Drivers Code** (em *For Developers*) → **+ New Driver**
2. Clique em **Import**, cole a URL abaixo e confirme:
   ```
   https://raw.githubusercontent.com/tecnosimples/hubitat_samsung_tv/main/drivers/SamsungTVRemote.groovy
   ```
3. Clique em **Save**.

Siga para a [etapa 3](#3-criar-e-configurar-o-device).

---

## 3. Criar e configurar o device

1. **Devices** → **Add Device** → **Virtual**
2. **Device Name:** o nome da TV (ex.: `TV Sala`)
3. **Type:** `Samsung TV Remote (TecnoSimples)` → **Save Device**
4. **Com a TV ligada**, em *Preferences*:
   - **Samsung TV Ip:** o IP fixo da etapa 1.2
   - Clique em **Save Preferences**
5. **Autorize o Hubitat na TV:** no primeiro comando, a TV mostra o pedido
   *"Permitir que o dispositivo se conecte?"*. Aceite no controle remoto em até 30 s.

### Testar

- Clique em **Off** no device: a TV desliga e o atributo `switch` vai para **off** em alguns segundos.
- Espere **30 s** (a TV entra em repouso profundo) e clique em **On**: a TV liga pelo Wake-on-LAN.
- Se as duas coisas funcionaram, está pronto.

---

## 🔌 Ligar e desligar: comando discreto

No Hubitat, **`on` sempre deixa a TV ligada e `off` sempre deixa desligada**. Repetir o comando não
inverte o estado: uma regra que dispara duas vezes não desliga a TV que acabou de ligar.

Por baixo, a TV só aceita a tecla **POWER**, que é *toggle* (a mesma tecla liga e desliga), e o
**Wake-on-LAN**, que só liga. O driver torna o comando discreto lendo o estado da TV antes de agir:

| Estado lido da TV | `on` | `off` |
|---|---|---|
| ligada | não faz nada | tecla POWER |
| em standby | tecla POWER | não faz nada |
| dormindo (não responde) | Wake-on-LAN | não faz nada |

- O resultado só é publicado depois de **confirmado pela TV**. Sem confirmação em 15 s, o driver
  avisa no log e publica o estado real.
- A tecla nunca é repetida antes de 5 s, para não ligar e desligar em seguida.
- **Exceção:** TVs que não informam o estado (alguns modelos antigos). Nelas o `off` manda uma tecla só,
  sem confirmar o standby. Não verificado em equipamento.

---

## ❓ Problemas comuns

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| Liga logo depois de desligar, mas não liga depois de alguns minutos | "Ligar com Celular" desativado | Etapa 1.1 |
| Nada funciona de repente | IP da TV mudou | Etapa 1.2 e corrija o IP no device |
| A TV não mostrou o pedido de autorização, ou ele foi negado | Hubitat bloqueado na TV | Na TV: **Configurações → Geral → Gerenciador de Dispositivos Externos → Gerenciador de Conexão de Dispositivos → Lista de Dispositivos**, libere ou apague o Hubitat e tente de novo |
| Não liga depois de faltar energia | Algumas TVs desligam o Wake-on-LAN até serem ligadas uma vez | Ligue uma vez pelo controle remoto |
| Às vezes não liga pelo Wi-Fi | Wake-on-LAN pelo Wi-Fi é menos confiável | Use cabo de rede (etapa 1.3) |

Ainda com problema? Ative **Enable debug logging for 30 minutes** no device (Save Preferences), repita o comando e envie o log (**Logs** → filtre pelo device).
