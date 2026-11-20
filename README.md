# 🚏 Ponto Certo

**Saia de casa na hora certa e pegue o ônibus sem esperar no ponto.**

O Ponto Certo é um aplicativo web para quem anda de ônibus em São Paulo. Ele usa os dados em tempo real
da **API Olho Vivo da SPTrans** para mostrar quando cada ônibus vai passar no seu ponto e calcular,
a partir do tempo que você leva a pé, a hora exata de sair de casa.

🔗 **Acesse o app:** https://SEU-PROJETO.vercel.app

> **Global Solution 2026.2** · Application Development · 2º ano de Ciência da Computação · Prof. Rodrigo Moraes
> Área de desafio: *Citizen Intelligence & Predictive Analytics*

---

## O problema

Quem depende de ônibus raramente sabe quando ele vai passar. Saindo cedo demais, a pessoa fica parada
no ponto, muitas vezes à noite, no frio ou na chuva. Saindo tarde, perde o ônibus e espera o próximo.
Para pessoas com deficiência, ainda há outra incerteza: saber se o próximo veículo é acessível.

## A solução

O Ponto Certo junta a **previsão de chegada** da SPTrans com o **tempo de caminhada até o ponto**
informado pelo usuário e responde a pergunta que importa: *"quando eu saio?"*

| Funcionalidade | Descrição |
|---|---|
| 🔎 **Busca de pontos** | Por rua ou nome do ponto, com opção de ordenar do mais perto ao mais longe pela localização do celular |
| 🕒 **Painel do ponto** | Próximos ônibus de cada linha, com horário e minutos até a chegada, atualizado automaticamente a cada 30 s |
| 🚶 **Hora de sair** | Para cada ônibus, o app indica **"Saia em X min"**, **"Saia agora!"** ou **"Não dá tempo"**, e destaca o melhor ônibus para pegar |
| 🔔 **Me avise** | Alerta por linha: na hora de sair, o app mostra um aviso, vibra, toca um bipe e envia uma notificação do navegador |
| ⭐ **Meus pontos** | Pontos favoritos na tela inicial, já com o próximo ônibus ou a hora de sair |
| 🚌 **Busca de linhas** | Por número ou destino (ex.: `8000`, `Lapa`), com escolha do sentido |
| 🗺️ **Onde está meu ônibus** | Mapa com o trajeto da linha e a posição de cada ônibus em tempo real (atualizada a cada 15 s); tocar em um ponto abre o painel dele |
| ♿ **Mobilidade acessível** | Modo com fonte grande e alto contraste, filtro "só veículos acessíveis" e destaque dos ônibus acessíveis no painel e no mapa |

A interface foi pensada primeiro para o celular, com botões grandes e fáceis de tocar.

## Endpoints da API Olho Vivo utilizados

| Endpoint | Uso no app |
|---|---|
| `POST /Login/Autenticar` | Autenticação com o token (feita no servidor) |
| `GET /Parada/Buscar` | Busca de pontos por nome ou endereço |
| `GET /Previsao/Parada` | Painel do ponto, hora de sair, alertas e resumo dos favoritos |
| `GET /Linha/Buscar` | Busca de linhas e identificação dos dois sentidos |
| `GET /Parada/BuscarParadasPorLinha` | Trajeto da linha no mapa |
| `GET /Posicao/Linha` | Posição dos ônibus em tempo real |

## Arquitetura

```
Navegador ──► /api/olhovivo (função serverless no Vercel) ──► api.olhovivo.sptrans.com.br/v2.1
          ◄── JSON                                         ◄── JSON
```

O front-end não chama a SPTrans diretamente. Todas as requisições passam por uma **função serverless**
(`api/olhovivo.js`) hospedada no Vercel, que:

- **autentica** na API com o token guardado em variável de ambiente, sem expô-lo no código público;
- **guarda o cookie de sessão** e o reaproveita, autenticando de novo automaticamente se a sessão expirar;
- **resolve o bloqueio de CORS** da API, já que a requisição parte do servidor;
- **aceita apenas os endpoints usados pelo app**, para não funcionar como um proxy aberto;
- **mantém as respostas em cache por alguns segundos** na CDN, reduzindo a carga sobre a API.

### Cálculo da hora de sair

A API retorna `hr` (hora da consulta) e, para cada veículo, `t` (hora prevista de chegada ao ponto):

```
minutos até chegar = t − hr              (considerando a virada da meia-noite)
sair em            = minutos até chegar − minutos de caminhada

sair em < 0   →  "Não dá tempo"
sair em ≤ 1   →  "Saia agora!"
caso contrário →  "Saia em {sair em} min"
```

O cálculo usa a hora informada pela própria API, e não o relógio do aparelho, para evitar erros
quando o relógio do celular está adiantado ou atrasado.

## Estrutura do projeto

```
ponto-certo/
├── index.html              # estrutura das telas: início, painel do ponto e mapa da linha
├── css/style.css           # estilos mobile first e modo acessível
├── js/
│   ├── app.js              # ponto de entrada: navegação entre telas e modo acessível
│   ├── api.js              # funções de acesso à API (via /api/olhovivo)
│   ├── previsao.js         # regras de negócio: organização das previsões e hora de sair
│   ├── alertas.js          # alertas "Me avise"
│   ├── mapa.js             # mapa com Leaflet
│   ├── armazenamento.js    # favoritos e preferências (localStorage)
│   ├── utils.js            # funções auxiliares (tempo, distância, segurança de HTML)
│   └── telas/
│       ├── inicio.js       # busca e "Meus pontos"
│       ├── ponto.js        # painel de previsões do ponto
│       └── linha.js        # mapa da linha em tempo real
├── api/olhovivo.js         # função serverless: proxy autenticado da API Olho Vivo
├── dev/                    # servidor local e dados de exemplo (não publicados no Vercel)
├── INTEGRANTES.txt         # integrantes do grupo e responsabilidades
└── package.json
```

A navegação acontece sem recarregar a página, usando o `#` do endereço (`#ponto/{codigo}` e
`#linha/{codigo}`). Assim, cada ponto ou linha tem um link próprio que pode ser salvo ou compartilhado.

## Tecnologias

- **HTML, CSS e JavaScript** puros (módulos ES), sem frameworks e sem etapa de build
- **[Leaflet](https://leafletjs.com)** com mapas do **[OpenStreetMap](https://www.openstreetmap.org)**
- APIs do navegador: Fetch, LocalStorage, Geolocation, Notifications, Vibration e Web Audio
- **Vercel** para hospedagem e função serverless em Node.js

## Executar localmente

Requisito: [Node.js](https://nodejs.org) 20 ou superior. Não há dependências para instalar.

```bash
git clone https://github.com/nicolasaraujo04/ponto-certo.git
cd ponto-certo
cp .env.example .env     # preencha SPTRANS_TOKEN com um token da SPTrans
npm run dev              # http://localhost:3000
```

Sem `SPTRANS_TOKEN`, o servidor local usa dados de exemplo no mesmo formato da API, o que permite
testar a interface sem acesso à SPTrans.

Para publicar no Vercel, importe o repositório e defina a variável de ambiente `SPTRANS_TOKEN`.

## Equipe

Os integrantes e as responsabilidades de cada um estão em [`INTEGRANTES.txt`](INTEGRANTES.txt).

## Créditos

- Dados de transporte em tempo real: **SPTrans – API Olho Vivo**
- Mapas: © colaboradores do **OpenStreetMap**
