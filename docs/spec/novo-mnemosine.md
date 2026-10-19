Mnemosine

Nova versão e o que será feito na mesma.
## A nova versão terá 2 domínios:
### Plugins/conectores
Neste domínio terá todo o código necessário para iteragir com os players (hermes-agent, claude-code, devin, codex, antigravity), a idéia é que a configuração relacionada ao plugin esteja toda nesse domínio.

Algumas das configurações que estarão sobre esse domínio:

RECALL (busca automática antes de cada prompt)
  QCTX_RECALL_DISABLED          "1" desliga o recall automático.
  QCTX_RECALL_STRICT_FLOOR      piso de score denso quando não há cross-encoder (padrão 0.58).
  QCTX_RECALL_DENSE_FLOOR       piso de score denso na 1ª etapa, quando há cross-encoder (padrão 0.45).
  QCTX_RECALL_MIN_SCORE         corte do cross-encoder na 2ª etapa (padrão 0.10).
  QCTX_RECALL_TOP_K             hits por ângulo de consulta no Qdrant (padrão 20 com cross-encoder, 8 sem).
  QCTX_RECALL_MAX_MEMORIES      máx. de memórias injetadas por prompt (padrão 6).
  QCTX_RECALL_MAX_CHARS         orçamento total de caracteres do bloco injetado (padrão 14000).
  QCTX_RECALL_MAX_PER_MEM       teto de caracteres por memória — a sobra é truncada (padrão 4500).
  QCTX_RECALL_BREAKER           cooldown do breaker do rerank em segundos (padrão 300; 0 desliga).
  QCTX_RECALL_QDRANT_BUDGET     segundos totais para TODAS as chamadas Qdrant de uma execução (padrão 5.0).

CHECKPOINT (injeção periódica do procedimento de escrita)
  QCTX_CHECKPOINT_DISABLED      "1" desliga o checkpoint.
  QCTX_CHECKPOINT_INTERVAL      interações entre checkpoints; vence o valor do arquivo de config (padrão 5; 0 desliga).

GUARDA DE ARQUIVO GRANDE (recusa leitura que estouraria a janela)
  QCTX_BIGFILE_ESCAPE           marcador de texto que, presente na chamada, força a leitura (padrão "--full").
  QCTX_BIGFILE_FLOOR_PCT        fração da janela mínima que deve sobrar depois da leitura (padrão 0.20).
  QCTX_BIGFILE_SHARE_PCT        fração do contexto livre que a leitura pode consumir sozinha (padrão 0.40).
  (os dois últimos também via qctx config set bigfile-floor-pct / bigfile-share-pct)

ESTADO E DAEMON
  QCTX_STATE_DIR                diretório do estado (janela de recall por sessão, breaker, recall.log, contadores); tem prioridade sobre o padrão (~/.mnemosine/state no claude-code; $HERMES_HOME/mnemosine-state no hermes).
  QCTX_DAEMON_AUTOSTART_DISABLED "1" impede o host de autoiniciar o daemon de indexação.

Só no hermes
  QCTX_HERMES_STATE_DB          caminho explícito do state.db do hermes (fonte do modelo/janela da sessão); padrão $HERMES_HOME/state.db ou ~/.hermes/state.db.

Configurações de conexão com o o domínio memory hub:

QDRANT
  QCTX_QDRANT_URL               endereço do Qdrant (legado: QDRANT_URL; padrão: vazio — obrigatório).
  QCTX_QDRANT_API_KEY           chave de API do Qdrant (legados: QDRANT_SERVICE_API_KEY, QDRANT_API_KEY; padrão: vazio). Segredo: config set recusa; só ambiente.

ENDPOINTS DE MODELOS
  QCTX_API_BASE_URL             base comum; derive embed_url (<base>/embeddings) e rerank_url (<base>/rerank) quando estes estão vazios (legado: SERVER_BASE_URL; padrão: vazio).
  QCTX_API_KEY                  chave para essa base; segredo, só ambiente (legado: SERVER_API_KEY; padrão: vazio).
  QCTX_EMBED_URL                URL completa do endpoint de embeddings; vence api_base_url (legado: RECALL_EMBED_URL; padrão: vazio).
  QCTX_RERANK_URL               URL completa do endpoint de rerank; vence api_base_url (legado: RECALL_RERANK_URL; padrão: vazio).
  QCTX_EMBED_MODEL              nome do modelo de embedding enviado no request (legado: EMBEDDING_MODEL; padrão: bge-m3).
  QCTX_RERANK_MODEL             nome do modelo de rerank enviado no request (legado: RECALL_RERANK_MODEL; padrão: bge-reranker-v2-m3).

COLEÇÕES (os cinco precisam ser distintos entre si — colisão é rejeitada)
  QCTX_MEMORY_COLLECTION        acervo de memórias curadas (legado: COLLECTION_NAME; padrão: vazio — obrigatório).
  QCTX_DOCS_COLLECTION          acervo de documentos temporários, destruíveis por limpeza (legado: DOCS_COLLECTION; padrão: memories_docs_tmp).
  QCTX_LIBRARY_COLLECTION       acervo de documentos permanentes (legado: LIBRARY_COLLECTION; padrão: memories_docs_library).
  QCTX_REPOS_COLLECTION         acervo de chunks de código indexado de repositórios (legado: REPOS_COLLECTION; padrão: memories_repos).
  QCTX_REPOS_REGISTRY_COLLECTION registro dos repositórios declarados, separado dos chunks (legado: REPOS_REGISTRY_COLLECTION; padrão: memories_repos_registry).

DIMENSÃO DE VETOR
  QCTX_VECTOR_SIZE              dimensão dos vetores gravados no Qdrant; precisa casar com o modelo de embedding (legado: VECTOR_SIZE; padrão: 1024).

JANELA DE CONTEXTO
  QCTX_CONTEXT_WINDOW           tamanho fixo da janela do modelo, em tokens (padrão: 0). Último recurso da guarda de arquivo grande — só vale quando o host não publicou o valor dinâmico e nem o cache do endpoint respondeu; 0 desliga o uso do valor fixo.
  
### Memory hub
Aqui é onde se tem um overview da memoria configurada e também dá a opção ao usuário de gerenciar a mesma.