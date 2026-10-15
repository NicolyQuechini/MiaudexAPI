# Miaudex 🐱

Aplicativo Flutter que consome a [The Cat API](https://thecatapi.com) para descobrir
fotos de gatos, conhecer raças, votar, favoritar e enviar as próprias fotos.

Trabalho acadêmico — construído a partir do wireframe de 7 telas e da
*Documentação Técnica da The Cat API* (Luana Sudatti e Nicoly Quechini).

> Pendências e pontos ainda não validados: veja [PENDENCIAS.md](PENDENCIAS.md).

---

## Sumário

1. [Funcionalidades](#funcionalidades)
2. [Tecnologias](#tecnologias)
3. [Como rodar](#como-rodar)
4. [Chave da API](#chave-da-api)
5. [Gerar o APK (Android)](#gerar-o-apk-android)
6. [Estrutura do projeto](#estrutura-do-projeto)
7. [Arquitetura](#arquitetura)
8. [Telas e endpoints](#telas-e-endpoints)
9. [Modelos de dados](#modelos-de-dados)
10. [Tratamento de erros](#tratamento-de-erros)
11. [Decisões de projeto](#decisões-de-projeto)
12. [Limitações do plano Free](#limitações-do-plano-free)
13. [Solução de problemas](#solução-de-problemas)
14. [Créditos e termos de uso](#créditos-e-termos-de-uso)

---

## Funcionalidades

| # | Tela | O que faz |
|---|---|---|
| 1 | **Onboarding** | Apresentação em 3 slides. "Começar" cria o usuário (nome e e-mail opcionais); "Já tenho uma conta" entra com um código existente. |
| 2 | **Descobrir** | Busca por raça, filtro por categoria (Chapéus, Espaço...), "Fato do dia" e grade de fotos com rolagem infinita. Coração favorita a foto. |
| 3 | **Raças** | Lista de raças com busca por nome e filtros **Calmas**, **Ativas** e **Sociáveis**. |
| 4 | **Detalhe da raça** | Foto, país de origem, expectativa de vida, descrição e barras de Afeto, Energia, Inteligência e Sociabilidade (1 a 5). Botão "Ver fotos desta raça". |
| 5 | **Votar** | Pilha de fotos: deslizar ou usar os botões. ✕ = voto -1, ⭐ = voto +1, ❤ = favorita e vota +1. Mostra os 4 últimos votos. |
| 6 | **Favoritos** | Abas **Todos**, **Por raça** e **Recentes** (últimos 7 dias). Tocar no coração remove. |
| 7 | **Perfil** | Contadores de votos/favoritos/fotos, "Minhas fotos" com situação (Aprovada, Em análise, Recusada), enviar e **apagar** fotos, "Meus votos" e "Sair da conta". |

Telas internas (fora das 7 do wireframe): fotos de uma raça, "Meus votos" e "Minhas fotos" (ver todas).
Todas as telas, exceto Descobrir (tela inicial), têm seta de voltar.

---

## Tecnologias

* Flutter / Dart (SDK `^3.5.0`)
* [`http`](https://pub.dev/packages/http) e [`http_parser`](https://pub.dev/packages/http_parser) — chamadas REST e envio de imagem (multipart)
* [`shared_preferences`](https://pub.dev/packages/shared_preferences) — guarda o usuário e as fotos enviadas no aparelho
* [`image_picker`](https://pub.dev/packages/image_picker) — escolher foto na galeria
* [`cached_network_image`](https://pub.dev/packages/cached_network_image) — cache das fotos

Sem pacote de gerenciamento de estado: usa `ChangeNotifier` + `ListenableBuilder` do próprio Flutter.

---

## Como rodar

**Pré-requisitos:** Flutter instalado (`flutter doctor` sem erros para a plataforma usada).

```bash
git clone <URL-DO-REPOSITORIO>
cd cat_app
flutter pub get
flutter run -d windows        # ou: flutter run  (celular/emulador Android conectado)
```

O app foi executado em **Windows** e em **celular Android**. Rodar no **Chrome (web)** não é
suportado: o navegador bloqueia imagens e chamadas de outros domínios (CORS).

---

## Chave da API

O app precisa de uma chave da The Cat API (gratuita) criada em <https://thecatapi.com>.
Sem chave válida, as **buscas de fotos funcionam**, mas votos, favoritos, categorias e envio de fotos
retornam **401**.

A chave é lida em `lib/config.dart` e **nunca deve ser enviada ao GitHub**. Escolha uma opção:

**Opção A — por linha de comando (recomendada)**

```bash
flutter run --dart-define=CAT_API_KEY=live_sua_chave
```

**Opção B — arquivo `env.json` (não versionado)**

```json
{"CAT_API_KEY": "live_sua_chave"}
```

```bash
flutter run --dart-define-from-file=env.json
```

Adicione `env.json` ao `.gitignore`.

**Opção C — direto no código (só para testes locais)**

Troque `'DEMO-API-KEY'` por a sua chave no `defaultValue` de `lib/config.dart`.
**Volte para `'DEMO-API-KEY'` antes de fazer commit.**

Observações:
* O valor passado por `--dart-define` sempre vence o `defaultValue`.
* A chave é limpa automaticamente (espaços e aspas sobrando são removidos).
* Depois de mudar a chave, feche e abra o app de novo (hot reload não atualiza).
* A chave fica dentro do APK gerado e pode ser extraída: não divulgue o APK publicamente.
  Em produção, use um servidor intermediário.

---

## Gerar o APK (Android)

1. Confirme que `android/app/src/main/AndroidManifest.xml` tem, antes de `<application`:
   ```xml
   <uses-permission android:name="android.permission.INTERNET"/>
   ```
   (no modo debug ela já vem; no release é preciso declarar).
2. Gere o APK:
   ```bash
   flutter clean
   flutter pub get
   flutter build apk --release --dart-define=CAT_API_KEY=live_sua_chave
   ```
3. O arquivo fica em `build/app/outputs/flutter-apk/app-release.apk`. Copie para o celular e instale
   (autorize "instalar apps de fontes desconhecidas"). Desinstale versões antigas antes.

**iOS:** adicione `NSPhotoLibraryUsageDescription` ao `ios/Runner/Info.plist` para o envio de fotos
(iOS não foi testado).

---

## Estrutura do projeto

```
lib/
├── main.dart                     Ponto de entrada: onboarding ou abas, conforme o usuário
├── config.dart                   Nome do app, URL base e chave da API
├── data/
│   └── cat_facts_pt.dart         Curiosidades em português (Fato do dia)
├── theme/
│   └── app_theme.dart            Cores e tema do wireframe
├── model/                        Classes Dart com fromJson
│   ├── json_utils.dart           Leitura segura de JSON (asInt, asBool, asDate...)
│   ├── cat_image.dart            Foto (ImageDto)
│   ├── breed.dart                Raça (ProBreedDto) e BreedImage
│   ├── vote.dart                 Voto (VoteDto)
│   ├── favourite.dart            Favorito (FavouriteDto)
│   ├── cat_category.dart         Categoria
│   ├── cat_colour.dart           Cor/padrão
│   ├── cat_fact.dart             Curiosidade
│   ├── health_tip.dart           Dica de saúde
│   └── my_photo.dart             Foto enviada + situação (Aprovada/Em análise/Recusada)
├── services/
│   ├── cat_api_client.dart       Única classe que fala com a API
│   └── cat_api_exception.dart    Exceção própria + mensagens para o usuário
├── state/
│   ├── app_state.dart            Usuário, favoritos, votos e fotos enviadas
│   └── navigation.dart           Aba selecionada (compartilhada)
├── screens/
│   ├── onboarding_screen.dart    Tela 1
│   ├── home_shell.dart           5 abas + barra de navegação (cada aba com seu Navigator)
│   ├── discover_screen.dart      Tela 2
│   ├── breeds_screen.dart        Tela 3
│   ├── breed_detail_screen.dart  Tela 4
│   ├── breed_photos_screen.dart  Fotos de uma raça
│   ├── vote_screen.dart          Tela 5
│   ├── favourites_screen.dart    Tela 6
│   ├── profile_screen.dart       Tela 7
│   ├── my_votes_screen.dart      Meus votos
│   └── my_photos_screen.dart     Minhas fotos (ver todas)
└── widgets/                      Componentes reutilizados (cards, chips, barra, botões...)
```

---

## Arquitetura

```
Screens/Widgets  →  AppState (ChangeNotifier)  →  CatApiClient  →  The Cat API
                         │
                         └→ SharedPreferences (usuário e ids das fotos enviadas)
```

* **model/** — converte o JSON em objetos. Só `id` e `name` são obrigatórios nas raças; os demais
  campos são anuláveis. Números passam por `asInt` (a API pode mandar inteiro ou decimal).
* **services/** — `CatApiClient` concentra todas as chamadas, envia `x-api-key` em todas as
  requisições, valida o status e lança `CatApiException`.
* **state/** — `AppState` é um *singleton* com o `sub_id` do usuário, a lista de favoritos, os votos
  e os ids das fotos enviadas. Também guarda em cache a lista de raças.
* **screens/** — telas; escutam o `AppState` com `ListenableBuilder`.
* **Navegação** — `HomeShell` usa um `Navigator` por aba, assim a barra de baixo continua visível
  ao abrir o detalhe de uma raça. O botão voltar do Android fecha telas internas, depois volta
  para Descobrir e só então sai do app.

### Usuário (`sub_id`)

A The Cat API não tem login. No primeiro uso o app gera um código único (`gato-xxxxxxxxxx`),
enviado como `sub_id` em votos, favoritos e envios. Ele aparece no Perfil (toque para copiar) e
permite recuperar votos e favoritos em outro aparelho com "Já tenho uma conta".

---

## Telas e endpoints

| Tela | Endpoints |
|---|---|
| Descobrir | `GET /images/search`, `GET /breeds/search`, `GET /categories` |
| Raças | `GET /breeds` (cache em memória) |
| Detalhe / Fotos da raça | dados de `/breeds`, `GET /images/search?breed_ids=` |
| Votar | `GET /images/search`, `POST /votes`, `POST /favourites` |
| Favoritos | `GET /favourites?attach_image=true`, `POST /favourites`, `DELETE /favourites/{id}` |
| Perfil | `GET /votes`, `GET /favourites`, `POST /images/upload`, `GET /images/{id}`, `GET /images/{id}/status`, `DELETE /images/{id}` |

**Situação das fotos enviadas** (`/images/{id}/status`):

| Resposta da API | Exibido no app |
|---|---|
| `clean`, `labelled` | Aprovada |
| `to_be_reviewed`, `flagged` | Em análise |
| `unsafe`, `does_not_contain_species` | Recusada |

**Envio de foto:** `multipart/form-data` com `file` e `sub_id`. O app descobre o tipo da imagem
(JPEG, PNG, GIF ou WebP) pelos primeiros bytes e envia o `content-type` correto; sem isso a API
responde "That file type is not supported".

---

## Modelos de dados

Seguem a seção 5 da documentação técnica: `CatImage`, `Breed`, `Vote`, `Favourite`, `CatFact`,
`HealthTip`, `CatCategory` e `CatColour`, todos com construtor `fromJson`. `CatFact`, `HealthTip` e
`CatColour` existem por completude, mas não são usados em nenhuma tela atualmente.

---

## Tratamento de erros

`CatApiException` guarda o código de status e o corpo da resposta, e traduz para o usuário:

| Situação | Mensagem |
|---|---|
| Sem conexão / tempo esgotado | "Sem conexão com a internet..." |
| 401 | "Chave da API ausente ou inválida" + começo e tamanho da chave em uso |
| 403 | Motivo devolvido pela API (ex.: recurso indisponível no plano Free) |
| 404 | "Não encontramos o que você procurava." |
| 429 | "Limite de requisições da API atingido." |

Falhas de API também são impressas no console (`ERRO API ...`) com o início da chave em uso,
o que ajuda a diagnosticar chave errada.

---

## Decisões de projeto

* **Economia de cota:** lista de raças em cache; abas só carregam quando abertas pela primeira vez;
  `include_breeds=true`; várias fotos por chamada; fotos em cache de disco.
* **Fato do dia:** o endpoint `/facts/random` retorna **403 (indisponível no plano Free)**. O app usa
  uma lista própria de curiosidades em português (`lib/data/cat_facts_pt.dart`), uma por dia.
* **Filtros de raça (Calmas / Ativas / Sociáveis):** usam os níveis de 1 a 5 (`energy_level`,
  `social_needs`, `stranger_friendly`) quando a API envia. Contas Free podem receber o formato
  simples da raça, sem esses números; nesse caso o filtro usa as palavras do campo `temperament`
  (ex.: "Active", "Energetic").
* **Categorias** vêm da API; o chip "Engraçados" do wireframe não é uma categoria da API e não aparece.
* **Coração na tela Votar** favorita a foto e também registra voto +1.
* **"Recentes" nos Favoritos** = salvos nos últimos 7 dias.
* **Fotos enviadas** são acompanhadas pelos ids guardados no aparelho (a API não oferece uma
  listagem simples das fotos de um usuário).
* **Nome do app:** `AppConfig.appName` em `lib/config.dart`.

---

## Limitações do plano Free

* 10.000 requisições por mês (contadas desde o primeiro uso da chave, inclusive em testes).
* `GET /facts/*` retorna 403 (ver acima).
* Campos extras de raça podem não vir (formato `BasicBreedDto`); as barras do detalhe mostram
  "sem dados" quando o valor não existe.
* A análise automática das fotos enviadas pode recusar fotos que mostram claramente um gato
  (falso positivo); a decisão é da API.

---

## Solução de problemas

| Sintoma | Causa provável e solução |
|---|---|
| Quase tudo vermelho no VS Code | Imports com o nome de pacote errado ou dependências não baixadas. O pacote é `cat_app`; rode `flutter pub get` e recarregue a janela. |
| "Chave da API ausente ou inválida" | A chave não chegou ao app (veja o texto "Chave em uso" na mensagem). Confira a [chave](#chave-da-api), rode de novo e desinstale APK antigo. |
| Fotos aparecem, mas votos/favoritos/envio dão 401 | A busca de fotos funciona mesmo com chave inválida; os demais endpoints não. |
| App no celular abre, mas não carrega nada | Falta a permissão `INTERNET` no `AndroidManifest.xml`. |
| "That file type is not supported" ao enviar foto | Use a versão atual do `cat_api_client.dart` (envia o `content-type`) e fotos em JPG, PNG, GIF ou WebP. |
| Nenhuma raça nos filtros | A conta recebe raças sem os níveis numéricos; o filtro usa o `temperament` (ver decisões). |
| No Chrome as fotos/dados não carregam | CORS do navegador. Rode no Windows ou Android. |
| `curl` no PowerShell dá erro de `Headers` | No Windows, use `curl.exe` em vez de `curl`. |

---

## Créditos e termos de uso

* Dados e imagens: [The Cat API](https://thecatapi.com) (That API Company).
  Consulte os [Termos de Uso](https://thecatapi.com/terms) antes de qualquer uso comercial.
* Projeto acadêmico, sem fins comerciais.
