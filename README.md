# Samsung TV Remote — TecnoSimples (Hubitat Elevation)

Driver local para TVs **Samsung Tizen** (2016+): controle remoto por WebSocket (portas 8001/8002),
apps, presets e ligar pela rede (Wake-on-LAN). Integração SmartThings opcional.

Baseado no **Samsung TV Remote** de Dave Gutheinz (Apache-2.0), com o **ligar/desligar reescrito**:

| Estado lido da TV (`PowerState`) | Ligar | Desligar |
|---|---|---|
| `on` | nada a fazer | tecla POWER, confirma `standby` |
| `standby` | tecla POWER (só 3 s ou mais depois do desligar) | nada: já está desligada |
| não responde (dormindo) | só Wake-on-LAN | nada: já está desligada |

O resultado só é publicado com o estado **lido da TV**; sem confirmação em 15 s, o driver avisa no log e
publica o estado real. Nada de "ligada por suposição".

Detalhes da regra (2.4.1 e 2.4.2):
- **Uma leitura perdida não decide.** O desligar e a sonda periódica só tratam a TV como desligada depois de
  duas leituras seguidas sem resposta.
- **A tecla POWER só se repete 5 s depois de ter saído**, para não ligar e desligar em seguida.
- **TV que responde sem informar `PowerState`** (modelos anteriores): conta como **ligada** enquanto responde.
  Não foi verificado num equipamento assim. Nela o desligar manda uma tecla só, porque o repouso não pode ser
  confirmado, e o dispositivo passa a aparecer "on" quando a TV responde (antes aparecia sempre "off").

📘 **Passo a passo completo (TV, HPM, sem HPM, problemas comuns): [GUIA_INSTALACAO.md](GUIA_INSTALACAO.md)**

## ✅ Requisito na TV

Ative **Ligar com Celular** (*Power On with Mobile*): Configurações → Geral → Rede → Configurações Avançadas.
Sem ele a TV não acorda pela rede. Use IP fixo (reserva DHCP) para a TV.

## 🚀 Instalação via Hubitat Package Manager (HPM)

1. No HPM: **Install** → **From a URL**.
2. Cole:
   ```
   https://raw.githubusercontent.com/tecnosimples/hubitat_samsung_tv/main/packageManifest.json
   ```
3. **Devices** → **Add Device** → **Virtual** → Type **`Samsung TV Remote (TecnoSimples)`**.
4. Preencha **Samsung TV Ip** e clique em **Save Preferences** com a TV **ligada**.
5. No primeiro comando, a TV mostra um pedido de autorização: aceite no controle remoto (expira em 30 s).

*Já usa o driver original do DaveGut?* Instale este pacote e troque o **Type** do device para
`Samsung TV Remote (TecnoSimples)`; depois **Save Preferences** com a TV ligada.

## ⚠️ Limites conhecidos

- Tempos medidos em uma TV (UN32T4300, Tizen 2020). A decisão é pelo estado lido, não por tempo, mas
  outros modelos podem variar.
- TV sem energia na tomada e religada: o Wake-on-LAN pode não funcionar até ela ser ligada uma vez pelo controle.

## Licença e créditos

Apache License 2.0 — ver [LICENSE](LICENSE). Obra modificada a partir de
[DaveGut/HubitatActive](https://github.com/DaveGut/HubitatActive/tree/master/SamsungTvRemote)
(Copyright 2022 Dave Gutheinz), com ajustes de hhorigian e da TecnoSimples (descritos no cabeçalho do driver).
