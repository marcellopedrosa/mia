# MIA — Marcell Inteligência Artificial

> **Arquitetura de identidade vocal proprietária, conversação e canto com independência de fornecedor**
>
> Status: arquitetura inicial / blueprint técnico  
> Nome da artista virtual: **Mia**  
> Significado: **Marcell Inteligência Artificial**  
> Objetivo principal: preservar, reproduzir e evoluir uma identidade vocal consistente de longo prazo, de forma que Mia continue sendo a mesma vocalista mesmo que os modelos, provedores de GPU, LLMs ou ferramentas de IA mudem.

---

## 1. Visão

Mia é uma vocalista virtual projetada para possuir uma identidade vocal persistente.

O objetivo deste projeto não é simplesmente "gerar uma voz parecida" em cada execução. O objetivo é criar um **ativo vocal versionado**, suficientemente bem documentado e preservado para que possa ser reutilizado durante anos em:

- músicas;
- backing vocals;
- demos;
- chamadas;
- vídeos;
- conversas em tempo real;
- entrevistas fictícias;
- apresentações;
- conteúdo de redes sociais;
- futuras aplicações ainda não previstas.

A premissa central é:

> **A identidade da Mia não pode pertencer a um serviço de terceiros.**

Serviços externos podem ser usados para:

- processamento intensivo;
- GPU;
- LLM;
- reconhecimento de fala;
- geração temporária de guias;
- treinamento;
- inferência;
- renderização.

Porém, os elementos necessários para reconstruir e reproduzir Mia devem permanecer sob controle do proprietário do projeto.

Isso inclui, no mínimo:

1. gravações-mestre;
2. dataset original;
3. dataset processado;
4. documentação da identidade;
5. evidências de origem e direitos;
6. checkpoints próprios;
7. configurações de treinamento;
8. versões dos modelos-base;
9. dependências;
10. containers ou ambientes reproduzíveis;
11. amostras de referência;
12. testes de regressão vocal;
13. hashes dos artefatos;
14. histórico de releases.

---

# 2. Princípio fundamental: identidade não é modelo

A arquitetura deve separar:

```text
IDENTIDADE DA ARTISTA
        │
        ├── dados canônicos
        ├── referências vocais
        ├── características fonéticas
        ├── estilo de fala
        ├── estilo de canto
        ├── pronúncia
        ├── persona
        └── direitos/proveniência
                │
                ▼
         MODELOS DE EXECUÇÃO
                │
       ┌────────┴────────┐
       │                 │
       ▼                 ▼
  FALA / TTS        CANTO / SVC
       │                 │
       └────────┬────────┘
                ▼
              MIA
```

Um modelo é apenas uma implementação da identidade.

Ao longo dos anos, Mia poderá usar:

```text
2026 -> Chatterbox / RVC
2027 -> outro TTS / outro SVC
2028 -> modelo multimodal unificado
2029 -> nova tecnologia
2030 -> outro runtime
```

A identidade canônica deve continuar existindo independentemente dessas mudanças.

---

# 3. O que realmente preserva a voz

Há três níveis diferentes de preservação.

## 3.1 Nível 1 — descrição

Exemplo:

```yaml
voice:
  gender: female
  apparent_age: 27
  texture: warm
  breathiness: medium
  register: mezzo-soprano
```

Isso é útil, mas **não preserva o timbre**.

Milhares de vozes poderiam corresponder à mesma descrição.

---

## 3.2 Nível 2 — dataset e referências

Aqui já existe uma representação real da identidade.

Exemplo:

```text
identity/master/
├── speech-neutral.wav
├── speech-soft.wav
├── speech-emotional.wav
├── singing-low.wav
├── singing-mid.wav
├── singing-high.wav
└── vowels-sustained.wav
```

Com material suficiente, futuros modelos poderão ser treinados novamente aproximando a identidade.

Entretanto:

> dataset não garante reprodução matemática idêntica de uma versão antiga.

Um modelo futuro poderá interpretar os dados de forma diferente.

---

## 3.3 Nível 3 — checkpoint + runtime reproduzível

Para preservar a versão exata de uma implementação da Mia, deve-se guardar:

```text
dataset
+
checkpoint
+
modelo-base
+
versão do código
+
dependências
+
configuração
+
container
+
seed quando aplicável
```

Por isso, cada release de Mia deve ser tratada como um artefato de software.

Exemplo:

```text
MIA-SPEECH-1.0.0
MIA-SING-1.0.0
```

---

# 4. Arquitetura de alto nível

```text
                              ┌─────────────────────┐
                              │      USUÁRIO        │
                              │ microfone / texto   │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  INPUT / ORCHESTR.  │
                              │      local          │
                              └──────────┬──────────┘
                                         │
                       ┌─────────────────┴─────────────────┐
                       │                                   │
                       ▼                                   ▼
               ┌───────────────┐                   ┌───────────────┐
               │     STT       │                   │     TEXT      │
               │ local/cloud   │                   │    direto     │
               └───────┬───────┘                   └───────┬───────┘
                       │                                   │
                       └─────────────────┬─────────────────┘
                                         ▼
                              ┌─────────────────────┐
                              │     AGENT CORE      │
                              │   persona + memória │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │      LLM API        │
                              │ Ollama Cloud / etc. │
                              └──────────┬──────────┘
                                         │
                                         ▼
                                      TEXTO
                                         │
                    ┌────────────────────┴─────────────────────┐
                    │                                          │
                    ▼                                          ▼
         ┌──────────────────────┐                   ┌──────────────────────┐
         │     SPEECH ENGINE    │                   │    SINGING ENGINE    │
         │ Chatterbox / plugin  │                   │ RVC / SVC / plugin   │
         └──────────┬───────────┘                   └──────────┬───────────┘
                    │                                          │
                    │                                          │
                    ▼                                          ▼
           MIA-SPEECH MODEL                           MIA-SING MODEL
                    │                                          │
                    └────────────────────┬─────────────────────┘
                                         ▼
                                 IDENTIDADE MIA
```

---

# 5. Separação obrigatória de responsabilidades

O projeto deve possuir interfaces independentes para:

```text
LLMProvider
SpeechToTextProvider
SpeechSynthProvider
SingingProvider
VoiceConversionProvider
StorageProvider
GPUProvider
```

Nenhuma camada deve assumir permanentemente um fornecedor.

Exemplo conceitual:

```python
class LLMProvider:
    def generate(self, messages):
        ...

class SpeechSynthProvider:
    def synthesize(self, text, voice_identity):
        ...

class VoiceConversionProvider:
    def convert(self, source_audio, voice_model):
        ...
```

Assim:

```text
OLLAMA -> pode ser substituído
GPU Provider -> pode ser substituído
STT -> pode ser substituído
TTS -> pode ser substituído
SVC -> pode ser substituído

MIA -> permanece
```

---

# 6. O ativo principal: MIA Identity Capsule

Criar um diretório chamado:

```text
mia-identity/
```

Ele representa a identidade canônica da artista.

Estrutura recomendada:

```text
mia-voice/
│
├── README.md
│
├── docs/
│   ├── architecture/
│   │   ├── overview.md
│   │   ├── speech.md
│   │   ├── singing.md
│   │   ├── storage.md
│   │   └── disaster-recovery.md
│   │
│   ├── adr/
│   │   ├── ADR-0001-voice-identity-ownership.md
│   │   ├── ADR-0002-speech-singing-separation.md
│   │   ├── ADR-0003-cloud-compute-local-ownership.md
│   │   ├── ADR-0004-master-audio-format.md
│   │   └── ADR-0005-model-versioning.md
│   │
│   └── legal/
│       ├── provenance.md
│       ├── licenses.md
│       └── consent/
│
├── identity/
│   ├── identity.yaml
│   ├── personality.yaml
│   ├── pronunciation.yaml
│   ├── languages.yaml
│   ├── singing-style.yaml
│   ├── speech-style.yaml
│   └── canonical-tests.yaml
│
├── dataset/
│   ├── raw/
│   │   ├── speech/
│   │   └── singing/
│   │
│   ├── curated/
│   │   ├── speech/
│   │   └── singing/
│   │
│   ├── processed/
│   │   ├── speech/
│   │   └── singing/
│   │
│   ├── metadata/
│   │   ├── speech.csv
│   │   ├── singing.csv
│   │   └── provenance.csv
│   │
│   └── manifests/
│
├── masters/
│   ├── speech/
│   ├── singing/
│   ├── phonetics/
│   └── canonical/
│
├── models/
│   ├── speech/
│   │   ├── v1/
│   │   └── current -> v1
│   │
│   ├── singing/
│   │   ├── v1/
│   │   └── current -> v1
│   │
│   └── base-models/
│
├── environments/
│   ├── docker/
│   ├── conda/
│   ├── pip/
│   └── lockfiles/
│
├── services/
│   ├── api/
│   ├── speech/
│   ├── singing/
│   ├── conversation/
│   └── orchestration/
│
├── evaluation/
│   ├── speech/
│   ├── singing/
│   ├── regression/
│   └── golden-samples/
│
├── releases/
│
├── scripts/
│   ├── dataset/
│   ├── backup/
│   ├── training/
│   ├── inference/
│   ├── evaluation/
│   └── integrity/
│
├── infra/
│   ├── docker/
│   ├── cloud-gpu/
│   └── terraform/
│
├── .env.example
├── .gitignore
└── Makefile
```

---

# 7. identity.yaml

Exemplo inicial:

```yaml
schema_version: 1

artist:
  id: MIA
  name: Mia
  full_name: Marcell Inteligência Artificial
  type: virtual_singer

identity:
  version: 1.0.0
  status: development

voice:
  perceived_gender: female
  apparent_age:
    target: 27
    acceptable_range: [24, 31]

  register:
    speaking: mezzo
    singing: mezzo_soprano

  timbre:
    warmth: 0.82
    brightness: 0.46
    breathiness: 0.32
    raspiness: 0.08
    softness: 0.72
    resonance: 0.66

speech:
  pace: medium_slow
  articulation: clear
  emotional_baseline: calm
  intimacy: medium_high

languages:
  primary: pt-BR
  secondary:
    - en

accent:
  portuguese:
    base: brazilian
    regionality: subtle
  english:
    type: international_brazilian
    strength: subtle

singing:
  primary_genres:
    - afro_house
    - melodic_house
    - organic_house

  preferred_bpm:
    min: 118
    max: 126
    center: 123

  interpretation:
    intimate: 0.75
    energetic: 0.45
    airy: 0.60
    dramatic: 0.35

ownership:
  canonical_dataset_required: true
  checkpoint_required: true
  vendor_voice_id_forbidden_as_source_of_truth: true
```

Os números não são uma descrição científica absoluta.

Eles existem para:

- documentação;
- comparação;
- regressão;
- consistência criativa;
- futuras migrações.

---

# 8. A regra mais importante do projeto

Nunca utilizar como fonte canônica algo como:

```text
voice_id = "vendor-xy-12345"
```

Isso pode ser usado como implementação temporária.

Nunca como identidade oficial.

A fonte de verdade deve ser:

```text
MIA IDENTITY CAPSULE
```

contendo:

```text
masters
dataset
metadata
checkpoints
configs
containers
tests
provenance
```

---

# 9. Como criar a voz original da Mia

Esta é a etapa mais importante.

## 9.1 O que evitar

Evitar construir Mia a partir de:

- voz de celebridade;
- voz extraída de música comercial;
- voz de cantora encontrada na internet;
- dataset sem licença clara;
- modelo de terceiros cujo contrato proíba re-treinamento;
- serviço que não permita exportar resultados;
- voz sintetizada por fornecedor cujo contrato não permita usar outputs como dataset.

---

## 9.2 Caminho recomendado: voz-base com direitos claros

O método mais robusto juridicamente e tecnicamente é:

```text
VOICE TALENT / FONTE AUTORIZADA
            │
            ▼
  contrato de uso para IA
            │
            ▼
      gravação original
            │
            ▼
       dataset MIA
```

O contrato deve prever explicitamente, conforme aplicável:

- gravação;
- treinamento de modelos;
- fine-tuning;
- voice conversion;
- TTS;
- singing voice;
- geração de obras derivadas;
- uso comercial;
- uso mundial;
- prazo;
- reutilização futura;
- armazenamento;
- criação de checkpoints;
- uso após encerramento da prestação;
- possibilidade de migrar entre tecnologias.

Consultar profissional jurídico para contrato definitivo.

---

# 10. Alternative Voice Design

Também é possível iniciar com uma voz gerada artificialmente.

Porém, antes de transformá-la em dataset proprietário, verificar:

```text
1. licença do modelo;
2. licença dos pesos;
3. termos do serviço;
4. direito comercial sobre output;
5. permissão para treinamento em outputs;
6. restrições de voice cloning;
7. restrições sobre modelos derivados.
```

Guardar uma cópia da licença vigente na data de criação:

```text
docs/legal/licenses/
├── source-license-2026-09-27.pdf
├── source-terms-2026-09-27.pdf
└── provenance.md
```

Não depender apenas de links, porque termos online podem mudar.

---

# 11. Gravações-mestre

Os masters são o DNA acústico de Mia.

Sugestão de formato de arquivo para preservação:

```text
WAV
PCM
24-bit
48 kHz
mono
sem compressão com perdas
```

O dataset de treinamento poderá posteriormente ser convertido para:

```text
16 kHz
22.05 kHz
24 kHz
40 kHz
44.1 kHz
48 kHz
```

dependendo do modelo.

Nunca substituir os masters pelos arquivos convertidos.

---

# 12. RAW deve ser imutável

Estrutura:

```text
dataset/raw/
```

Regra:

> arquivos RAW nunca são alterados.

Correções devem gerar derivados.

Exemplo:

```text
RAW
mia_session_001_take_003.wav

CURATED
mia_session_001_take_003_trimmed.wav

PROCESSED
mia_session_001_take_003_40k.wav
```

---

# 13. Dataset de fala

Para uma identidade robusta, gravar diferentes categorias.

## 13.1 Fala neutra

- frases curtas;
- frases longas;
- números;
- datas;
- perguntas;
- afirmações;
- exclamações;
- nomes próprios;
- abreviações;
- siglas.

---

## 13.2 Cobertura fonética

Criar frases que cubram:

- vogais;
- consoantes;
- encontros consonantais;
- nasalização;
- ditongos;
- diferentes posições silábicas;
- fonemas raros.

Para PT-BR incluir variedade suficiente para:

```text
r
rr
lh
nh
ch
j
s
z
ão
õe
ãe
```

---

## 13.3 Emoções

Criar subset:

```text
neutral
soft
happy
sad
intimate
energetic
serious
whisper-like
```

Não misturar emoção sem metadados.

---

# 14. Dataset de canto

O dataset de canto deve ser separado do dataset de fala.

```text
dataset/raw/singing/
```

Registrar:

- vocal seco;
- sem reverb;
- sem delay;
- sem autotune destrutivo;
- sem backing track vazando;
- sem compressão excessiva;
- sem limiter pesado.

Guardar também uma versão totalmente crua.

---

## 14.1 Cobertura musical

A gravação deve cobrir:

```text
notas graves
notas médias
notas agudas
transições de registro
sustain
staccato
legato
ataques suaves
ataques fortes
vibrato
sem vibrato
falsete, se fizer parte da identidade
```

---

## 14.2 Vogais sustentadas

Registrar, em diferentes notas:

```text
A
E
I
O
U
```

com variações de intensidade.

Isso ajuda a documentar:

- formantes;
- ressonância;
- vibrato;
- estabilidade;
- comportamento do timbre por registro.

---

## 14.3 Frases específicas para Afro House

Criar corpus com:

- frases curtas;
- hooks;
- repetições;
- vocalizações;
- chamadas;
- palavras prolongadas;
- respirações;
- entradas fora do tempo forte;
- frases de baixa densidade silábica.

Exemplo estrutural:

```text
Verse
short phrase
silence
short phrase

Build
repetition

Hook
Mia...
Mia...
```

Não é necessário limitar o dataset apenas a um gênero.

A identidade precisa ser mais ampla que uma música.

---

# 15. Metadados do dataset

Exemplo:

```csv
id,file,type,language,style,pitch_min,pitch_max,quality,approved
SP001,speech/001.wav,speech,pt-BR,neutral,,,A,true
SG001,singing/001.wav,singing,pt-BR,intimate,A3,E5,A,true
```

Adicionar também:

```text
session
microphone
preamp
room
distance_cm
date
engineer
source
rights
notes
sha256
```

---

# 16. Proveniência

Cada arquivo precisa responder:

```text
Quem gravou?
Quando?
Com qual autorização?
De onde veio?
Pode ser usado comercialmente?
Pode treinar IA?
Pode gerar derivados?
```

Exemplo:

```yaml
asset_id: SG001
source_type: original_recording
performer: CONTRACT-ID-001
consent_document: consent/CONTRACT-ID-001.pdf
ai_training_allowed: true
commercial_allowed: true
derivative_models_allowed: true
```

---

# 17. Canonical Voice Masters

Separar aproximadamente 10 a 30 amostras consideradas representativas.

```text
masters/canonical/
├── speech-neutral-01.wav
├── speech-neutral-02.wav
├── speech-soft-01.wav
├── speech-emotional-01.wav
├── singing-low-01.wav
├── singing-mid-01.wav
├── singing-high-01.wav
├── singing-hook-01.wav
└── sustained-vowels.wav
```

Esses arquivos serão utilizados em testes de regressão.

---

# 18. Arquitetura de fala

Pipeline:

```text
MICROFONE
    │
    ▼
VAD
    │
    ▼
STT
    │
    ▼
AGENT CORE
    │
    ▼
LLM REMOTO
    │
    ▼
TEXTO
    │
    ▼
MIA SPEECH ENGINE
    │
    ▼
ÁUDIO
```

---

# 19. LLM não faz parte da identidade vocal

O LLM representa:

```text
raciocínio
conversa
conhecimento
resposta
```

A voz representa:

```text
identidade acústica
```

Portanto:

```text
LLM = substituível
MIA Voice = ativo persistente
```

Hoje:

```text
Ollama Cloud
```

Amanhã poderia ser:

```text
outro provider
```

sem alterar o timbre de Mia.

---

# 20. Ollama remoto

Ollama pode atuar como provider remoto de LLM.

O Agent Core deve chamar uma interface semelhante a:

```text
POST /llm/chat
```

e receber:

```json
{
  "text": "Oi Marcello, como você está?"
}
```

Esse texto é enviado ao Speech Engine.

---

# 21. Speech Engine recomendado

Arquitetura sugerida:

```text
interface SpeechEngine
        │
        ├── ChatterboxAdapter
        ├── F5Adapter
        └── FutureAdapter
```

## 21.1 Chatterbox

Bom candidato inicial para experimentação porque possui:

- implementação open source;
- voice cloning por referência;
- versão multilíngue;
- português;
- modelo específico PT-BR nas releases recentes;
- licença permissiva na distribuição oficial consultada.

Ainda assim:

> guardar versão exata do código, pesos e licença utilizada.

Nunca confiar apenas em `pip install latest`.

---

## 21.2 F5-TTS

Pode ser mantido como adapter experimental.

Atenção:

```text
código -> MIT
pesos oficiais pré-treinados -> CC-BY-NC
```

Isso significa que o uso comercial dos pesos oficiais requer análise cuidadosa.

Não considerar F5-TTS como fundamento comercial sem revisar licença do checkpoint específico utilizado.

---

# 22. Fala sem GPU local

A ausência de GPU local não impede propriedade da identidade.

Arquitetura:

```text
PC LOCAL
   │
   │ envia temporariamente
   │ checkpoint + input
   ▼
GPU REMOTA
   │
   │ inferência
   ▼
WAV
   │
   ▼
PC LOCAL
```

A GPU é apenas computação.

A propriedade deve estar nos arquivos:

```text
local storage
external backup
private repository metadata
```

---

# 23. GPU efêmera

Estratégia recomendada para treinamento:

```text
1. criar GPU temporária;
2. provisionar container;
3. enviar dataset necessário;
4. treinar;
5. validar;
6. baixar checkpoints;
7. baixar logs;
8. baixar configs;
9. gerar manifest;
10. verificar hashes;
11. destruir ambiente remoto.
```

A máquina remota não deve ser o único local contendo:

```text
dataset
checkpoint
modelo
config
```

---

# 24. Bring Your Own Weights

Objetivo futuro:

```text
GPU_PROVIDER
      │
      ▼
docker image
      +
MIA checkpoint
      +
request
```

Nenhum provider deve "possuir Mia".

O runtime recebe Mia.

---

# 25. Arquitetura de canto

O canto deve ter pipeline independente.

Versão inicial:

```text
GUIDE VOCAL
    │
    ▼
clean / dry vocal
    │
    ▼
pitch / content extraction
    │
    ▼
MIA SING MODEL
    │
    ▼
VOICE CONVERSION
    │
    ▼
MIA VOCAL
    │
    ▼
mix / mastering
```

---

# 26. Por que Voice Conversion é útil

No estágio inicial, é melhor separar:

```text
PERFORMANCE
```

de:

```text
TIMBRE
```

Uma voz guia fornece:

- melodia;
- timing;
- respiração;
- dinâmica;
- vibrato;
- interpretação.

O modelo de Mia fornece:

- identidade vocal;
- timbre;
- características aprendidas.

Assim:

```text
GUIDE VOCAL
      +
MIA VOICE MODEL
      =
MIA CANTANDO
```

---

# 27. RVC como primeira implementação de canto

RVC — Retrieval-based Voice Conversion — pode ser usado inicialmente como motor de voice conversion.

Arquitetura:

```text
MIA-SING-1
   │
   ├── model.pth
   ├── feature.index
   ├── config.yaml
   ├── training-manifest.json
   ├── dependencies.lock
   └── docker-image.txt
```

Não armazenar apenas:

```text
model.pth
```

Sem o restante, a capacidade de reprodução diminui.

---

# 28. Guia vocal não precisa ser Mia

Esse ponto é estratégico.

A guia pode ser produzida por:

- você;
- outra pessoa autorizada;
- cantora de sessão;
- sintetizador;
- sistema de canto baseado em MIDI;
- ferramenta temporária externa.

A identidade final é aplicada depois.

Portanto, o gerador de guia pode ser substituído sem destruir a identidade.

```text
Guide Provider A
        │
        ▼
      GUIDE
        │
        ▼
  MIA SING ENGINE
        │
        ▼
       MIA
```

---

# 29. Futuro: síntese direta de canto

Em uma etapa posterior, poderá existir:

```text
MIDI
+
LYRICS
+
MIA SINGING MODEL
        │
        ▼
MIA CANTANDO DIRETAMENTE
```

Possíveis famílias de tecnologia:

- Singing Voice Synthesis;
- DiffSinger;
- OpenUtau + voicebank compatível;
- futuros modelos de canto.

Esse componente também deve ser adapter.

```text
SingingSynthesisProvider
```

---

# 30. Separar identidade e produção musical

Não colocar permanentemente no modelo de Mia:

```text
reverb
delay
chorus
EQ criativo
compressão pesada
mastering
```

Esses elementos pertencem ao projeto musical.

Pipeline correto:

```text
MIA DRY VOCAL
      │
      ▼
DAW
      │
      ├── EQ
      ├── Compression
      ├── De-esser
      ├── Delay
      ├── Reverb
      └── FX
```

Guardar sempre:

```text
dry vocal
wet vocal
mix stem
master
```

---

# 31. Speech e Singing são dois modelos diferentes

Manter:

```text
MIA-SPEECH
MIA-SING
```

Mesmo que futuramente um único modelo consiga fazer ambos.

Motivo:

```text
fala exige:
- naturalidade
- latência
- conversação

canto exige:
- pitch
- sustain
- vibrato
- dinâmica
- alcance
```

---

# 32. Versionamento semântico

Usar:

```text
MAJOR.MINOR.PATCH
```

Exemplo:

```text
MIA-SPEECH-1.0.0
MIA-SING-1.0.0
```

Mudança de PATCH:

```text
bugfix
ajuste sem alteração perceptível relevante
```

MINOR:

```text
melhoria mantendo identidade
```

MAJOR:

```text
mudança perceptível significativa
```

---

# 33. Voice Identity Version

Além do modelo:

```text
MIA-IDENTITY-1.0
```

Exemplo:

```yaml
identity: MIA-IDENTITY-1.0

speech_model:
  version: MIA-SPEECH-1.2.0

sing_model:
  version: MIA-SING-1.1.0
```

---

# 34. Release Manifest

Cada release deve gerar:

```json
{
  "artist": "MIA",
  "identity_version": "1.0.0",
  "model": "MIA-SING",
  "model_version": "1.0.0",
  "base_engine": "rvc",
  "engine_commit": "COMMIT_HASH",
  "dataset": "MIA-SING-DATASET-1.0",
  "created_at": "YYYY-MM-DD",
  "artifacts": [
    {
      "file": "model.pth",
      "sha256": "..."
    },
    {
      "file": "feature.index",
      "sha256": "..."
    }
  ]
}
```

---

# 35. SHA-256

Gerar hashes para todo artefato crítico.

Linux:

```bash
sha256sum model.pth
```

Python:

```python
from hashlib import sha256
from pathlib import Path

def checksum(path: str):
    h = sha256()

    with open(path, "rb") as f:
        for block in iter(lambda: f.read(1024 * 1024), b""):
            h.update(block)

    return h.hexdigest()

print(checksum("model.pth"))
```

---

# 36. Manifest de integridade

Criar:

```text
manifest.sha256
```

Exemplo:

```text
HASH  model.pth
HASH  feature.index
HASH  identity.yaml
HASH  training-config.yaml
```

Verificação:

```bash
sha256sum -c manifest.sha256
```

---

# 37. Git

Git deve armazenar:

```text
código
configuração
manifests
documentação
scripts
metadata
ADRs
schemas
testes
```

Git não deve necessariamente armazenar os arquivos enormes diretamente.

---

# 38. Git LFS

Pode ser usado para alguns binários.

Entretanto, não transformar GitHub no único repositório dos ativos.

Checkpoints e datasets grandes precisam de estratégia própria de armazenamento.

---

# 39. Storage

Arquitetura recomendada:

```text
                  MIA ASSETS
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
    WORK COPY      BACKUP A      BACKUP B
      SSD          external      off-site
```

Aplicar regra 3-2-1:

```text
3 cópias
2 tipos/localizações de mídia
1 cópia fisicamente separada
```

---

# 40. Independência inclusive da nuvem

A cópia off-site pode ser:

```text
HD externo armazenado em outro local
```

Portanto, nem o backup precisa depender de cloud.

Uma cloud criptografada pode ser adicional.

---

# 41. Criptografia

Dataset e checkpoints podem ser sensíveis.

Recomenda-se:

```text
disk encryption
encrypted archives
encrypted backup
```

Não colocar áudio RAW privado em repositório Git público.

---

# 42. Arquivo de release

Exemplo:

```text
releases/
└── MIA-SING-1.0.0/
    ├── manifest.json
    ├── manifest.sha256
    ├── model.pth
    ├── feature.index
    ├── config.yaml
    ├── environment.lock
    ├── Dockerfile
    ├── source-commit.txt
    ├── LICENSES/
    ├── evaluation/
    └── README.md
```

Compactar:

```text
MIA-SING-1.0.0.tar.zst
```

---

# 43. Container reproduzível

Exemplo conceitual:

```dockerfile
FROM nvidia/cuda:12.8.0-runtime-ubuntu24.04

WORKDIR /app

COPY requirements.lock .
RUN pip install -r requirements.lock

COPY src/ src/

ENTRYPOINT ["python", "-m", "mia.singing"]
```

Nunca depender somente de:

```dockerfile
FROM some-image:latest
```

Pin:

```text
version
digest
commit
```

sempre que possível.

---

# 44. Dependências

Guardar:

```text
requirements.txt
requirements.lock
poetry.lock
uv.lock
conda-lock.yml
```

conforme ferramenta escolhida.

Registrar também:

```text
Python
CUDA
cuDNN
PyTorch
FFmpeg
driver
```

---

# 45. Base models

Quando permitido pela licença, guardar localmente:

```text
models/base-models/
```

Caso não seja possível redistribuir publicamente:

```text
manifest + hash + origem
```

e cópia privada autorizada.

---

# 46. Ambiente sem GPU local

Separar tarefas.

## Local — CPU e storage

Rodar localmente:

```text
Git
manifests
hash
dataset catalog
metadata
backup
audio inspection
normalização não destrutiva
orquestração
UI
Agent Core
API
versionamento
testes leves
```

---

## Remoto — GPU

Executar remotamente:

```text
training
fine-tuning
TTS pesado
voice conversion pesado
singing synthesis
batch generation
embedding extraction pesado
```

---

# 47. Cloud compute não deve ser storage canônico

Regra:

```text
CLOUD GPU = WORKER
LOCAL STORAGE = SOURCE OF TRUTH
```

---

# 48. Fluxo de treinamento remoto

```text
LOCAL
  │
  ├── dataset release
  ├── training config
  ├── Docker image
  └── expected manifest
       │
       ▼
REMOTE GPU
       │
       ├── train
       ├── evaluate
       └── package
       │
       ▼
LOCAL
       │
       ├── verify hash
       ├── archive
       ├── backup
       └── tag release
```

---

# 49. Não fazer treinamento diretamente no dataset master

Antes de treinar:

```text
RAW
 │
 ▼
CURATED
 │
 ▼
DATASET RELEASE
 │
 ▼
TRAIN
```

Exemplo:

```text
MIA-SPEECH-DATASET-1.0.0
MIA-SING-DATASET-1.0.0
```

---

# 50. Dataset release

Estrutura:

```text
dataset/releases/
└── MIA-SING-DATASET-1.0.0/
    ├── audio/
    ├── metadata.csv
    ├── manifest.json
    ├── manifest.sha256
    └── README.md
```

---

# 51. Conversação

Agent Core:

```text
MiaAgent
```

Responsabilidades:

```text
persona
memória
contexto
prompt
LLM
tools
```

Não deve conter DSP ou TTS.

---

# 52. Personality

Exemplo:

```yaml
name: Mia

personality:
  calm: 0.75
  warm: 0.80
  artistic: 0.75
  humor: 0.35
  verbosity: 0.45

communication:
  default_language: pt-BR
  use_short_sentences: true
  avoid_excessive_formality: true
```

---

# 53. Persona e voz são coisas diferentes

Mia pode continuar com o mesmo timbre mesmo se sua personalidade mudar.

Ou manter a mesma personalidade com outro motor LLM.

```text
VOICE != PERSONALITY
```

---

# 54. Memória

A memória deve ficar separada do provider LLM.

Exemplo:

```text
memory/
├── profile/
├── conversations/
├── summaries/
└── embeddings/
```

Assim trocar o LLM não apaga o histórico.

---

# 55. API local

Criar serviço:

```text
mia-api
```

Endpoints iniciais:

```text
POST /v1/chat
POST /v1/speech
POST /v1/voice-conversion
POST /v1/singing
GET  /v1/identity
GET  /v1/models
GET  /v1/health
```

---

# 56. POST /v1/speech

Request:

```json
{
  "text": "Oi, meu nome é Mia.",
  "voice": "MIA-SPEECH-1.0.0",
  "style": "neutral",
  "language": "pt-BR"
}
```

Response:

```json
{
  "audio_id": "AUD-001",
  "model": "MIA-SPEECH-1.0.0",
  "file": "AUD-001.wav"
}
```

---

# 57. POST /v1/voice-conversion

```json
{
  "source": "guide.wav",
  "model": "MIA-SING-1.0.0",
  "pitch_shift": 0
}
```

---

# 58. Registry de modelos

Criar:

```yaml
models:

  speech:
    current: MIA-SPEECH-1.0.0

    versions:
      MIA-SPEECH-1.0.0:
        engine: chatterbox
        path: models/speech/v1

  singing:
    current: MIA-SING-1.0.0

    versions:
      MIA-SING-1.0.0:
        engine: rvc
        path: models/singing/v1
```

---

# 59. Adapter pattern

Estrutura Python:

```text
src/mia/
├── providers/
│   ├── llm/
│   │   ├── base.py
│   │   └── ollama.py
│   │
│   ├── speech/
│   │   ├── base.py
│   │   └── chatterbox.py
│   │
│   └── singing/
│       ├── base.py
│       └── rvc.py
```

---

# 60. Interface Speech

```python
from abc import ABC, abstractmethod

class SpeechProvider(ABC):

    @abstractmethod
    def synthesize(
        self,
        text: str,
        model_version: str,
        language: str,
        style: str
    ) -> bytes:
        pass
```

---

# 61. Interface Singing

```python
class SingingProvider(ABC):

    @abstractmethod
    def convert(
        self,
        source_audio: str,
        model_version: str,
        pitch_shift: float = 0
    ) -> str:
        pass
```

---

# 62. Provider remoto de GPU

Criar interface:

```python
class ComputeProvider:

    def submit(self, job):
        ...

    def status(self, job_id):
        ...

    def download(self, job_id):
        ...
```

Adapters:

```text
RunPodAdapter
VastAdapter
ModalAdapter
FutureProviderAdapter
```

Não colocar lógica de Mia dentro deles.

---

# 63. Job Manifest

Cada processamento remoto deve receber um job imutável.

```json
{
  "job_id": "JOB-20260927-001",
  "type": "voice_conversion",
  "model": "MIA-SING-1.0.0",
  "input_sha256": "...",
  "container_digest": "...",
  "parameters": {
    "pitch_shift": 0
  }
}
```

---

# 64. Output Manifest

```json
{
  "job_id": "JOB-20260927-001",
  "output": "mia-vocal.wav",
  "sha256": "...",
  "runtime": {
    "engine": "rvc",
    "version": "..."
  }
}
```

---

# 65. Observabilidade

Guardar:

```text
training loss
validation samples
GPU type
training duration
dataset version
model version
git commit
container digest
```

Não depender apenas de SaaS de experiment tracking.

Exportar logs para arquivos locais.

---

# 66. Experimentos

Cada experimento:

```text
experiments/
└── EXP-0001/
    ├── hypothesis.md
    ├── config.yaml
    ├── results.json
    ├── samples/
    └── conclusion.md
```

---

# 67. Golden Samples

Criar frases fixas.

Exemplo:

```text
GS-001:
"Oi, eu sou a Mia."

GS-002:
"Hoje vamos trabalhar em uma nova música."

GS-003:
"Recife encontra o mar enquanto a cidade dança."
```

Para canto:

```text
GC-001
melodia fixa + letra fixa

GC-002
mesma nota sustentada

GC-003
mesmo hook em diferentes versões
```

---

# 68. Teste A/B

Ao atualizar modelo:

```text
MIA-SPEECH-1.0
vs
MIA-SPEECH-1.1
```

gerar exatamente os mesmos prompts.

Comparar:

```text
timbre
pronúncia
ritmo
sotaque
idade percebida
expressividade
ruído
artefatos
```

---

# 69. Avaliação humana

Criar escala:

```text
1 = completamente diferente
2 = diferente
3 = aceitável
4 = muito semelhante
5 = identidade preservada
```

Avaliar por categoria.

---

# 70. Avaliação automática

Pode incluir:

```text
speaker embeddings
cosine similarity
pitch distribution
formant analysis
F0 statistics
duration
speech rate
spectral statistics
```

Nunca usar uma única métrica como verdade absoluta.

A identidade vocal é parcialmente perceptual.

---

# 71. Voice Drift

Definir:

```text
VOICE DRIFT
```

como qualquer alteração não intencional perceptível na identidade.

Exemplos:

```text
voz ficou mais jovem
sotaque mudou
voz ficou muito brilhante
rouquidão aumentou
respiração desapareceu
grave perdeu corpo
```

Toda release deve passar por regressão.

---

# 72. Canonical Acceptance Suite

Arquivo:

```text
identity/canonical-tests.yaml
```

Exemplo:

```yaml
tests:

  - id: SPEECH-NEUTRAL
    input: "Oi, eu sou a Mia."
    reference: masters/canonical/speech-neutral-01.wav

  - id: SPEECH-EN
    input: "Hi, I'm Mia."
    reference: masters/canonical/speech-english-01.wav

  - id: SING-HOOK
    source: evaluation/golden-samples/hook-guide.wav
    reference: masters/canonical/singing-hook-01.wav
```

---

# 73. Dataset Growth

Novas gravações nunca devem simplesmente sobrescrever a versão anterior.

Exemplo:

```text
MIA-SING-DATASET-1.0
MIA-SING-DATASET-1.1
MIA-SING-DATASET-2.0
```

---

# 74. Compatibilidade retroativa

Sempre manter a capacidade de executar:

```text
MIA-SING-1.0.0
```

mesmo após lançamento de:

```text
MIA-SING-2.0.0
```

quando tecnicamente possível.

Isso permite reproduzir músicas antigas.

---

# 75. Música como build reproduzível

Cada faixa pode possuir:

```text
songs/
└── recife/
    ├── song.yaml
    ├── lyrics.md
    ├── midi/
    ├── guides/
    ├── vocals/
    ├── stems/
    └── manifests/
```

---

# 76. song.yaml

```yaml
song:
  id: SONG-001
  title: Recife

tempo:
  bpm: 123

mia:
  identity: MIA-IDENTITY-1.0
  singing_model: MIA-SING-1.0.0

production:
  guide: guide-v3.wav
  final_dry_vocal: mia-dry-v1.wav
```

---

# 77. Reprodução histórica

Daqui a cinco anos será possível saber:

```text
qual Mia cantou?
qual checkpoint?
qual dataset?
qual pitch shift?
qual engine?
qual container?
qual guide vocal?
```

---

# 78. Git tags

Sugestão:

```text
mia-identity-v1.0.0
mia-speech-v1.0.0
mia-sing-v1.0.0
song-recife-v1.0
```

---

# 79. Branch strategy

Inicialmente:

```text
main
develop
```

Pode ser suficiente.

Artefatos binários não devem depender da branch.

---

# 80. .gitignore

Exemplo:

```gitignore
.env

# secrets
secrets/

# python
.venv/
__pycache__/
*.pyc

# generated audio
tmp/
output/

# raw proprietary assets
dataset/raw/

# checkpoints
*.pth
*.pt
*.ckpt
*.safetensors

# large local backups
backup/
```

Os arquivos críticos ignorados pelo Git devem existir em storage versionado externo ao repositório.

---

# 81. .env.example

```env
MIA_IDENTITY_VERSION=MIA-IDENTITY-1.0.0

LLM_PROVIDER=ollama
OLLAMA_BASE_URL=
OLLAMA_API_KEY=
OLLAMA_MODEL=

SPEECH_PROVIDER=chatterbox
MIA_SPEECH_MODEL=MIA-SPEECH-1.0.0

SINGING_PROVIDER=rvc
MIA_SING_MODEL=MIA-SING-1.0.0

GPU_PROVIDER=
GPU_API_KEY=

MIA_STORAGE=/data/mia
```

---

# 82. Secrets

Nunca commitar:

```text
API keys
tokens
passwords
cloud credentials
private contracts
private raw audio
```

---

# 83. Backups

Script sugerido:

```text
scripts/backup/backup_identity.sh
```

Backup mínimo:

```text
identity/
masters/
dataset/
models/
releases/
docs/legal/
evaluation/golden-samples/
environments/
```

---

# 84. Disaster Recovery

Cenário:

```text
computador principal destruído
GitHub indisponível
fornecedor de GPU encerrou
modelo original desapareceu
```

Mia deve continuar recuperável.

Checklist:

```text
[ ] repository clone
[ ] master dataset
[ ] raw dataset
[ ] curated dataset
[ ] model checkpoints
[ ] base model
[ ] Docker image/archive
[ ] dependencies
[ ] manifests
[ ] canonical samples
[ ] license records
[ ] encryption keys
```

---

# 85. Recovery Test

A cada grande release:

```text
1. usar máquina limpa;
2. restaurar backup;
3. reconstruir container;
4. carregar checkpoint;
5. executar golden samples;
6. comparar;
7. registrar resultado.
```

Backup não testado não é recuperação garantida.

---

# 86. Container archive

Além de guardar Dockerfile, quando relevante exportar imagem:

```bash
docker save mia-speech:1.0.0 | gzip > mia-speech-1.0.0.tar.gz
```

Guardar hash.

Isso reduz dependência de registries externos.

---

# 87. Python wheels

Para dependências críticas, pode-se guardar cache privado:

```text
vendor/wheels/
```

quando licenças permitirem.

Isso ajuda se algum pacote desaparecer.

---

# 88. Código de terceiros

Registrar:

```text
nome
versão
commit
licença
URL de origem
hash
```

Arquivo:

```text
THIRD_PARTY_MODELS.md
```

---

# 89. Licenças

Não assumir que:

```text
código open source == pesos open source
```

São artefatos diferentes.

Verificar:

```text
source code license
model weights license
dataset license
output license
commercial terms
```

---

# 90. Snapshot de licença

Para cada release:

```text
releases/MIA-SPEECH-1.0.0/LICENSES/
```

Guardar os textos das licenças aplicáveis.

---

# 91. Fala em tempo real

Arquitetura futura:

```text
Microphone
   │
   ▼
Local VAD
   │
   ▼
Streaming STT
   │
   ▼
LLM streaming
   │
   ▼
Sentence chunker
   │
   ▼
TTS streaming
   │
   ▼
Audio player
```

---

# 92. Latência

Prioridade da conversa:

```text
STT parcial
LLM streaming
TTS por chunks
playback incremental
```

Não esperar a resposta inteira quando não for necessário.

---

# 93. Interrupção

O agente deve permitir:

```text
user barge-in
```

Fluxo:

```text
Mia falando
     │
usuário começa a falar
     │
playback interrompido
     │
novo turno
```

---

# 94. Identidade conversacional

Além do timbre, guardar:

```text
ritmo de fala
pausas
energia
expressões
pronúncias preferidas
```

Exemplo:

```yaml
pronunciation:
  "Marcell": "mar-SELL"
  "Mia": "MI-a"
```

Para PT-BR, utilizar representação compatível com o engine quando existir suporte fonético.

---

# 95. Canto multilíngue

Criar datasets separados por idioma quando necessário:

```text
singing/pt-BR
singing/en
```

Avaliar voice drift entre idiomas.

---

# 96. Pronúncia musical

Em canto, a pronúncia pode precisar divergir da fala.

Exemplo:

```yaml
singing_pronunciation:
  rules:
    - word: Recife
      style: elongated_final_vowel
```

A implementação exata dependerá do motor.

---

# 97. Quality Gates

Nenhum modelo vira `current` automaticamente.

Critérios:

```text
[ ] dataset documented
[ ] legal provenance valid
[ ] canonical tests executed
[ ] no unacceptable voice drift
[ ] artifacts hashed
[ ] container reproducible
[ ] backup completed
[ ] recovery tested
```

---

# 98. Promotion

Fluxo:

```text
EXPERIMENTAL
   │
   ▼
CANDIDATE
   │
   ▼
VALIDATED
   │
   ▼
CURRENT
```

---

# 99. Model Registry local

Arquivo:

```text
registry/models.yaml
```

Pode futuramente migrar para MLflow ou outra ferramenta, mas o registro legível em arquivo continua sendo a fonte de verdade.

---

# 100. Hardware local mínimo

O projeto foi desenhado considerando máquina sem GPU potente.

Necessário localmente:

```text
CPU normal
RAM suficiente para tooling
SSD
microfone
backup externo
internet
```

GPU local é opcional.

---

# 101. Equipamento de gravação

Mais importante que GPU para a identidade inicial:

```text
bom microfone
ambiente silencioso
posição consistente
ganho consistente
pouco ruído
```

A qualidade do dataset limita a qualidade máxima da identidade.

---

# 102. Registro de sessão

Cada sessão deve ter:

```yaml
session_id: MIA-REC-001
date: YYYY-MM-DD

microphone:
  model: ""

interface:
  model: ""

room:
  description: ""

settings:
  sample_rate: 48000
  bit_depth: 24
  format: wav

distance_cm: 15
```

---

# 103. Não aplicar limpeza destrutiva

Evitar no RAW:

```text
noise gate agressivo
noise reduction forte
pitch correction
hard limiter
heavy compression
reverb
```

Produzir versões processadas separadamente.

---

# 104. Naming convention

Áudio:

```text
MIA_{TYPE}_{SESSION}_{TAKE}_{VERSION}.wav
```

Exemplo:

```text
MIA_SPEECH_S001_T003_RAW.wav
MIA_SING_S002_T014_RAW.wav
```

---

# 105. IDs imutáveis

Mesmo que arquivo seja renomeado, o asset deve possuir ID permanente.

```text
MIA-AUDIO-000001
```

---

# 106. Banco de metadados

Inicialmente:

```text
CSV/YAML/JSON
```

Posteriormente:

```text
SQLite/PostgreSQL
```

Não começar com infraestrutura complexa sem necessidade.

---

# 107. MVP técnico recomendado

## Fase 0 — governança

Criar:

```text
repository
identity.yaml
ADRs
storage
backup
legal provenance
```

---

## Fase 1 — Voice Master

Objetivo:

> definir como Mia realmente soa.

Entregáveis:

```text
canonical speech samples
canonical singing samples
identity v1
```

Não treinar modelo definitivo antes disso.

---

## Fase 2 — Speech MVP

Stack inicial sugerida:

```text
UI local
       │
Agent Core Python
       │
Ollama Cloud
       │
Chatterbox PT-BR
       │
GPU sob demanda
```

Objetivo:

```text
conversar com Mia
```

---

## Fase 3 — Singing MVP

```text
guide vocal
       │
RVC
       │
MIA-SING-1.0
```

Objetivo:

```text
primeira música inteira
```

---

## Fase 4 — Evaluation

Criar:

```text
golden speech suite
golden singing suite
voice drift scorecard
```

---

## Fase 5 — Automação

Criar CLI:

```bash
mia speech "Oi, eu sou a Mia"

mia sing \
  --guide guide.wav \
  --song SONG-001

mia evaluate \
  --model MIA-SING-1.1.0
```

---

## Fase 6 — conversa em tempo real

Adicionar:

```text
microfone
VAD
STT
streaming
interruption
```

---

## Fase 7 — MIDI + lyrics

Adicionar Singing Synthesis Provider.

Objetivo:

```text
lyrics + MIDI -> guide -> Mia
```

ou futuramente:

```text
lyrics + MIDI -> Mia direto
```

---

# 108. CLI conceitual

```text
mia identity show

mia models list

mia models promote MIA-SPEECH-1.0.0

mia speech synth \
    --text "Oi Marcello" \
    --style neutral

mia sing convert \
    --input guide.wav \
    --model MIA-SING-1.0.0

mia integrity verify

mia backup create

mia recovery test
```

---

# 109. Makefile

Exemplo:

```makefile
identity-check:
	python scripts/integrity/check_identity.py

manifest:
	python scripts/integrity/create_manifest.py

speech:
	python -m mia.cli speech

sing:
	python -m mia.cli sing

backup:
	./scripts/backup/backup_identity.sh

verify:
	sha256sum -c manifest.sha256
```

---

# 110. API Gateway local

Opcionalmente usar:

```text
FastAPI
```

Arquitetura:

```text
FastAPI
  │
  ├── /speech
  ├── /sing
  ├── /chat
  ├── /models
  └── /identity
```

---

# 111. Estado da conversa

Não armazenar estado exclusivamente no LLM.

Manter:

```text
conversation_id
turns
memory references
persona version
LLM provider
model
```

---

# 112. Auditabilidade

Cada resposta de Mia pode gerar:

```json
{
  "conversation_id": "...",
  "llm_provider": "ollama",
  "llm_model": "...",
  "speech_model": "MIA-SPEECH-1.0.0",
  "identity": "MIA-IDENTITY-1.0.0"
}
```

---

# 113. Segurança contra uso indevido

Como o modelo representa uma identidade vocal reutilizável:

```text
checkpoints privados
dataset privado
APIs autenticadas
logs de acesso
rate limits
```

Não publicar checkpoints sem decisão explícita.

---

# 114. Marca d'água

Modelos futuros podem suportar watermark.

Mesmo assim, não depender exclusivamente dele.

Guardar metadados de geração internamente.

---

# 115. Catálogo de músicas

Cada vocal exportado deve poder ser ligado a:

```text
modelo
versão
input
música
data
```

---

# 116. Alterações artísticas

Caso a Mia evolua:

```text
MIA 1
MIA 2
```

não destruir a identidade anterior.

Exemplo:

```text
MIA-IDENTITY-1.x
MIA-IDENTITY-2.x
```

Assim músicas históricas permanecem reproduzíveis.

---

# 117. Continuidade após desaparecimento de uma tecnologia

Caso Chatterbox deixe de existir:

```text
masters
dataset
MIA-SPEECH checkpoint
container
source snapshot
```

permitem manter a implementação antiga enquanto um novo modelo é preparado.

Caso RVC desapareça:

```text
checkpoint
source
container
```

continuam disponíveis.

Paralelamente, o dataset pode treinar um sucessor.

---

# 118. O que deve ser preservado por décadas

Classificação:

## Tier A — insubstituível

```text
RAW recordings
contracts / provenance
canonical masters
identity specification
```

---

## Tier B — altamente importante

```text
curated datasets
dataset releases
trained checkpoints
feature indexes
golden samples
```

---

## Tier C — reproduzível, mas arquivar

```text
containers
dependencies
source snapshots
configs
```

---

## Tier D — descartável

```text
temporary GPU
cache
temporary outputs
scratch files
```

---

# 119. Backup Tier A

Ideal:

```text
Copy 1 -> workstation
Copy 2 -> encrypted external SSD/HDD
Copy 3 -> encrypted off-site disk
```

Cloud opcional:

```text
Copy 4 -> encrypted object storage
```

---

# 120. Encryption Key Recovery

Se backups forem criptografados:

```text
não guardar a única chave no computador principal
```

Manter procedimento de recuperação documentado e seguro.

---

# 121. Repositório público vs privado

Recomendação:

```text
código/documentação -> pode ser público
identity assets -> privado
models/checkpoints -> privado inicialmente
legal docs -> privado
```

Separar em dois repositórios pode ser útil:

```text
mia-core          -> código
mia-private-assets -> identidade
```

---

# 122. Estrutura multi-repo opcional

```text
mia-core/
mia-models-private/
mia-dataset-private/
mia-music/
```

Para MVP, um monorepo com storage externo também funciona.

---

# 123. Sugestão de arquitetura Git

```text
GitHub
  │
  ▼
mia-core
  │
  ├── docs
  ├── code
  ├── manifests
  └── metadata

LOCAL PRIVATE STORAGE
  │
  ├── raw dataset
  ├── masters
  └── checkpoints

BACKUP STORAGE
  │
  └── encrypted mirror
```

---

# 124. Design Goal: Vendor Exit

Para cada fornecedor, responder:

```text
Se essa empresa fechar amanhã, Mia continua existindo?
```

Se resposta for não:

> existe vendor lock-in.

---

# 125. Design Goal: Model Exit

Pergunta:

```text
Se esse modelo open source parar de ser mantido amanhã,
consigo continuar executando a versão atual?
```

Para responder sim, manter:

```text
source snapshot
weights
dependencies
runtime
container
```

---

# 126. Design Goal: Re-training

Pergunta:

```text
Se eu quiser treinar um modelo totalmente novo daqui a cinco anos,
tenho os dados necessários?
```

Para responder sim:

```text
raw masters
high-quality dataset
metadata
phonetic coverage
singing coverage
provenance
```

---

# 127. Design Goal: Artistic Consistency

Pergunta:

```text
Como sei se ainda é Mia?
```

Resposta:

```text
canonical samples
human evaluation
speaker similarity
voice profile
golden tests
release review
```

---

# 128. Definition of Done — identidade

MIA-IDENTITY-1.0 só existe quando:

```text
[ ] voice source legally established
[ ] raw masters archived
[ ] speech canonical samples approved
[ ] singing canonical samples approved
[ ] identity.yaml complete
[ ] provenance complete
[ ] backup A complete
[ ] backup B complete
[ ] SHA-256 manifests generated
```

---

# 129. Definition of Done — speech

MIA-SPEECH-1.0:

```text
[ ] model trained/configured
[ ] Portuguese approved
[ ] English test executed
[ ] canonical prompts generated
[ ] voice drift acceptable
[ ] checkpoint archived
[ ] base model archived/identified
[ ] dependencies locked
[ ] container archived
[ ] licenses archived
[ ] backup completed
```

---

# 130. Definition of Done — singing

MIA-SING-1.0:

```text
[ ] singing dataset release created
[ ] model trained
[ ] low register tested
[ ] middle register tested
[ ] upper register tested
[ ] sustained notes tested
[ ] vibrato tested
[ ] Afro House hook tested
[ ] Portuguese tested
[ ] checkpoint archived
[ ] index archived
[ ] environment reproducible
[ ] golden samples approved
[ ] backup completed
```

---

# 131. Ordem prática para iniciar

Não começar por infraestrutura excessiva.

## Passo 1

Criar repositório:

```text
mia-voice
```

---

## Passo 2

Criar:

```text
docs/
identity/
masters/
dataset/
models/
evaluation/
scripts/
```

---

## Passo 3

Criar ADR:

```text
ADR-0001
Mia's canonical identity belongs to the project,
not to any AI provider.
```

---

## Passo 4

Definir fonte legal da voz.

Essa decisão deve acontecer antes de treinar qualquer modelo definitivo.

---

## Passo 5

Produzir Voice Master.

---

## Passo 6

Gerar:

```text
MIA-IDENTITY-1.0.0
```

---

## Passo 7

Testar speech zero-shot/few-shot.

---

## Passo 8

Selecionar Speech Engine.

---

## Passo 9

Criar:

```text
MIA-SPEECH-0.1
```

---

## Passo 10

Conversar com Mia.

Testar:

```text
timbre
sotaque
pronúncia
naturalidade
emoção
```

---

## Passo 11

Ajustar corpus.

---

## Passo 12

Criar MIA-SPEECH-1.0.

---

## Passo 13

Criar dataset de canto.

---

## Passo 14

Treinar primeiro RVC/SVC.

---

## Passo 15

Criar MIA-SING-0.1.

---

## Passo 16

Produzir música piloto.

Sugestão:

```text
SONG-001
```

---

## Passo 17

Comparar Mia falando e cantando.

Pergunta:

```text
Parece a mesma pessoa?
```

---

## Passo 18

Ajustar.

---

## Passo 19

Publicar:

```text
MIA-IDENTITY-1.0
MIA-SPEECH-1.0
MIA-SING-1.0
```

---

# 132. Decisão inicial de stack

## Orquestração

```text
Python
FastAPI
Pydantic
```

---

## LLM

```text
Provider abstraction
```

Primeira implementação:

```text
Ollama Cloud
```

---

## STT

Primeira opção:

```text
Whisper
```

Pode ser local ou remoto.

STT não é identidade.

---

## Speech

Primeiro candidato:

```text
Chatterbox Multilingual / PT-BR
```

---

## Speech experimental

```text
F5-TTS
```

avaliando restrições de licença dos pesos utilizados.

---

## Singing voice conversion

Primeiro candidato:

```text
RVC
```

---

## Futuro singing synthesis

```text
OpenUtau / DiffSinger-compatible workflow
```

após validação de licença e esforço.

---

## GPU

```text
provider adapter
```

Começar com GPU sob demanda.

---

## Local storage

```text
filesystem
```

Depois:

```text
NAS/object storage
```

se necessário.

---

# 133. Por que Chatterbox primeiro

Objetivo do speech MVP:

```text
ouvir Mia conversando
```

Não necessariamente treinar imediatamente um modelo enorme.

Voice cloning por áudio de referência permite validar rapidamente a identidade antes de investir em fine-tuning.

---

# 134. Por que RVC primeiro para canto

Objetivo:

```text
validar timbre em música
```

RVC permite testar rapidamente:

```text
guide vocal -> Mia
```

antes de construir Singing Voice Synthesis completo.

---

# 135. Por que não depender de F5 como base comercial inicial

A distribuição oficial informa uma diferença relevante:

```text
code: MIT
pretrained weights: CC-BY-NC
```

O projeto pode continuar possuindo adapter F5.

Mas o checkpoint específico deve passar por avaliação de licença antes de uso comercial.

---

# 136. Direitos sobre o dataset são tão importantes quanto a tecnologia

Para Mia ser realmente um ativo sustentável, deve ser possível provar a origem do material.

Não basta tecnicamente possuir:

```text
.wav
```

É necessário possuir o direito de utilizá-lo na finalidade pretendida.

---

# 137. Política de dados

Sugestão:

```text
dataset/raw         -> append only
dataset/curated     -> versioned
dataset/processed   -> reproducible
models              -> immutable releases
masters             -> immutable
```

---

# 138. Dataset lineage

Exemplo:

```text
RAW-001
  │
  ▼
CURATED-014
  │
  ▼
DATASET-1.0
  │
  ├── MODEL-SPEECH-1.0
  └── MODEL-SPEECH-1.1
```

---

# 139. Model Card

Cada modelo deve possuir:

```text
MODEL_CARD.md
```

Informações:

```text
model ID
purpose
engine
dataset
training
limitations
languages
known issues
license
evaluation
```

---

# 140. Dataset Card

Cada dataset release:

```text
DATASET_CARD.md
```

Incluindo:

```text
source
duration
languages
recording conditions
processing
rights
exclusions
known issues
```

---

# 141. Limitações documentadas

Exemplo:

```text
MIA-SING-1.0

Known limitations:
- instability above C6
- English vowels less stable
- aggressive belting not approved
```

Melhor documentar do que esconder.

---

# 142. Histórico

CHANGELOG:

```text
CHANGELOG.md
```

Exemplo:

```text
MIA-SPEECH-1.1
- melhor pronúncia PT-BR
- reduziu brilho excessivo
- preservou canonical voice score
```

---

# 143. Reprodutibilidade de training

Guardar comando exato:

```text
train.sh
```

Exemplo:

```bash
python train.py \
  --dataset MIA-SING-DATASET-1.0 \
  --config configs/mia-sing-v1.yaml \
  --output runs/mia-sing-v1
```

---

# 144. Seed

Quando framework suportar:

```text
seed
```

deve ser registrada.

Não garante total determinismo em GPU, mas aumenta reprodutibilidade.

---

# 145. Snapshot de código

Guardar:

```text
git commit
```

e opcionalmente:

```text
source.tar.gz
```

---

# 146. Evolução tecnológica

Criar processo anual:

```text
MODEL TECHNOLOGY REVIEW
```

Perguntas:

```text
Existe motor melhor?
A licença é melhor?
A similaridade é melhor?
A latência melhorou?
A nova tecnologia consegue reproduzir Mia?
```

Migrar apenas após passar Golden Suite.

---

# 147. Nunca migrar por novidade

Novo modelo não significa automaticamente melhor Mia.

Critério principal:

```text
identidade + qualidade + sustentabilidade
```

---

# 148. Independence Score

Pode-se criar score interno:

```text
Identity Data Owned      YES
Checkpoint Owned         YES
Runtime Archived         YES
Vendor Required          NO
Base Model Archived      YES
License Archived         YES
Recovery Tested          YES
```

Objetivo:

```text
Vendor Required = NO
```

para releases canônicas.

---

# 149. Cenário ideal

```text
Internet caiu.
```

Você talvez não consiga usar LLM remoto ou GPU.

Mas ainda possui:

```text
Mia dataset
Mia masters
Mia models
Mia code
Mia containers
Mia history
```

Quando outra infraestrutura estiver disponível, Mia volta a executar.

---

# 150. O que significa "possuir Mia"

Neste projeto, possuir Mia significa controlar:

```text
IDENTITY
DATA
RIGHTS
MODELS
VERSION HISTORY
RECOVERY
```

Não significa necessariamente possuir fisicamente a GPU que executa inference.

---

# 151. Primeira meta concreta

Criar:

```text
MIA VOICE MASTER v1
```

antes de qualquer treinamento definitivo.

Conteúdo mínimo:

```text
10-30 canonical speech clips
10-30 canonical singing clips
phonetic samples
sustained vowels
identity.yaml
provenance
SHA-256 manifest
```

---

# 152. Segunda meta

Criar:

```text
MIA-SPEECH-0.1
```

e realizar uma conversa real.

Perguntas de teste:

```text
"Quem é você?"

"Fale Recife."

"Fale lentamente."

"Fale em inglês."

"Agora fale de forma mais íntima."

"Conte até vinte."
```

---

# 153. Terceira meta

Criar:

```text
MIA-SING-0.1
```

e usar exatamente a mesma referência identitária.

Testes:

```text
spoken phrase
same phrase sung
low note
mid note
high note
hook
```

---

# 154. Quarta meta

Gerar:

```text
SONG-001
```

com manifesto completo.

---

# 155. Resultado final esperado

Ao final da primeira versão:

```text
Mia fala.
Mia conversa.
Mia canta.
Mia possui uma identidade documentada.
Mia possui dataset.
Mia possui checkpoints.
Mia possui releases.
Mia pode trocar de LLM.
Mia pode trocar de GPU provider.
Mia pode trocar de motor no futuro.
Mia não depende de um voice_id externo.
```

---

# 156. Arquitetura resumida

```text
                     ┌────────────────────────┐
                     │   MIA IDENTITY CAPSULE │
                     │                        │
                     │ masters                │
                     │ dataset                │
                     │ provenance             │
                     │ identity spec          │
                     │ checkpoints            │
                     │ tests                  │
                     └───────────┬────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                    ▼                         ▼
             MIA SPEECH                  MIA SING
                    │                         │
              TTS ENGINE                 SVC ENGINE
                    │                         │
                    └────────────┬────────────┘
                                 │
                                 ▼
                                MIA
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
          ▼                      ▼                      ▼
       conversa                música                 mídia
          │
          ▼
       LLM API
     substituível
```

---

# 157. Princípios de arquitetura

1. **Identity first.**
2. **Dataset before vendor.**
3. **Local ownership, remote compute.**
4. **Cloud GPU is disposable.**
5. **Models are versioned artifacts.**
6. **Speech and singing are separate capabilities.**
7. **No external voice ID is canonical.**
8. **Every release must be reproducible.**
9. **Every critical artifact must have backup.**
10. **Every model upgrade must pass voice regression.**
11. **Rights and provenance are part of the architecture.**
12. **Mia must survive the disappearance of any individual provider.**

---

# 158. ADRs recomendados

Criar inicialmente:

```text
ADR-0001
Canonical Voice Identity Ownership

ADR-0002
Separation of Speech and Singing Models

ADR-0003
Local Ownership with Ephemeral Cloud Compute

ADR-0004
Canonical Audio Master Format

ADR-0005
Voice Dataset Versioning

ADR-0006
Model Artifact Versioning

ADR-0007
Vendor-neutral Provider Interfaces

ADR-0008
Voice Regression Test Strategy

ADR-0009
Backup and Disaster Recovery

ADR-0010
Voice Provenance and Consent
```

---

# 159. Roadmap

```text
PHASE 0
Governance

PHASE 1
Voice Design

PHASE 2
Voice Master

PHASE 3
Speech MVP

PHASE 4
Conversational Mia

PHASE 5
Singing Dataset

PHASE 6
Singing MVP

PHASE 7
First Song

PHASE 8
Evaluation Automation

PHASE 9
Cloud GPU Automation

PHASE 10
Singing Synthesis

PHASE 11
Production Platform
```

---

# 160. Referências técnicas iniciais

Estas tecnologias são referências de implementação, não partes permanentes da identidade.

## Chatterbox

Repository:

```text
https://github.com/resemble-ai/chatterbox
```

Uso pretendido:

```text
speech
voice cloning
multilingual TTS
PT-BR
```

---

## F5-TTS

Repository:

```text
https://github.com/SWivid/F5-TTS
```

Uso:

```text
speech experiment
```

Atenção às licenças dos checkpoints oficiais.

---

## RVC

Repository:

```text
https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI
```

Uso:

```text
voice conversion
singing MVP
```

---

## Whisper

Repository:

```text
https://github.com/openai/whisper
```

Uso:

```text
speech-to-text
```

Não faz parte da identidade.

---

## OpenUtau

Repository:

```text
https://github.com/openutau/openutau
```

Uso futuro possível:

```text
singing synthesis workflow
```

---

## DiffSinger family

Uso futuro possível:

```text
lyrics + notes/MIDI -> singing synthesis
```

Toda implementação deve passar por revisão específica de licença antes de ser incorporada.

---

# 161. Licenciamento — nota de engenharia

A situação de licenças de projetos e pesos pode mudar.

Antes de cada release de produção:

```text
VERIFY LICENSES
```

e salvar snapshot.

Não confiar somente neste README.

---

# 162. Decisão arquitetural central

> **Mia não é Chatterbox.**
>
> **Mia não é RVC.**
>
> **Mia não é Ollama.**
>
> **Mia não é uma GPU na nuvem.**
>
> Mia é o conjunto versionado de dados, referências, direitos, parâmetros, modelos, testes e histórico que definem a artista.

---

# 163. Definition of Success

O projeto será considerado bem arquitetado quando for possível fazer o seguinte:

```text
remover o provider LLM atual
        ↓
conectar outro
        ↓
Mia continua Mia


remover o provider de GPU
        ↓
subir outro
        ↓
Mia continua Mia


migrar de TTS
        ↓
treinar/adaptar novo motor
        ↓
passar Golden Suite
        ↓
Mia continua Mia


migrar de RVC
        ↓
novo SVC/SVS
        ↓
passar Golden Suite
        ↓
Mia continua Mia
```

---

# 164. Próximo passo operacional

A primeira implementação do projeto deve ser deliberadamente pequena.

Criar:

```text
mia-voice/
├── README.md
├── docs/adr/
├── identity/
│   └── identity.yaml
├── masters/
├── dataset/
├── models/
├── evaluation/
└── scripts/
```

Depois criar:

```text
ADR-0001
```

com a decisão:

> A identidade canônica de Mia será armazenada e versionada independentemente de qualquer fornecedor de IA. Serviços externos serão tratados exclusivamente como mecanismos substituíveis de computação, inferência ou geração auxiliar.

Somente depois iniciar a criação da voz.

---

# 165. Conclusão

A meta deste projeto não é apenas ter uma IA que canta.

A meta é construir uma **artista digital tecnicamente preservável**.

Mia deverá ter:

```text
voz
identidade
histórico
dataset
modelos
memória
personalidade
músicas
releases
```

e, principalmente:

```text
independência tecnológica.
```

O hardware pode mudar.

O LLM pode mudar.

A cloud pode mudar.

O framework pode mudar.

O motor de voz pode mudar.

A identidade da Mia deve permanecer.

---

## Estado inicial proposto

```yaml
project: MIA
meaning: Marcell Inteligência Artificial

identity:
  version: 0.1.0
  status: design

speech:
  preferred_initial_engine: chatterbox
  status: planned

singing:
  preferred_initial_engine: rvc
  status: planned

llm:
  architecture: provider_agnostic
  initial_candidate: ollama_cloud

compute:
  strategy: ephemeral_remote_gpu

ownership:
  canonical_assets: local
  vendor_lock_in: forbidden

backup:
  strategy: 3-2-1

primary_goal:
  "Mia must remain reproducible independently of any single external provider."
```

---

**Mia — Marcell Inteligência Artificial**

> Uma vocalista virtual deve ser tratada como um ativo de identidade, não como uma configuração de um fornecedor.
