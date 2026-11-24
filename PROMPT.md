# Prompt de desenvolvimento: Biblioteca de Louvor

Cole o texto abaixo no Claude Code (ou outra IA de programação) dentro da pasta do projeto.

Antes de começar, prepare:
- PDFs em `documentos/louvor`, com a data no nome (ex.: `2026-11-16.pdf`).
- Ícone da igreja em `assets/logo-iel.png`.
- Telas da Opção A em `referencias/` (arquivos `01-entrar.png` a `09-exportar-pdf.png`, já incluídos no repositório).
- E-mails Google da equipe e link da playlist do YouTube.

---

Você é meu desenvolvedor web sênior. Vamos construir, do zero e em etapas, a "Biblioteca de Louvor" da Igreja Evangélica Livre: app online, instalável no celular (PWA), com cifras, detecção de tom, transposição e cadastro colaborativo. Eu não sou desenvolvedor experiente, então explique cada configuração com passos simples e diga exatamente onde clicar. Antes de codar, confirme o que entendeu e liste o que falta me perguntar.

=== 1. DECISÕES JÁ TOMADAS ===
- Backend: Firebase (Authentication com Google, Firestore, Hosting). Sem servidor próprio.
- Front-end: HTML, CSS e JavaScript puros (módulos ES), sem framework e sem etapa de build. Arquivos: index.html, styles.css, app.js (dividido em módulos: auth, db, transpose, pdf, ui, settings), sw.js, manifest.webmanifest. Código comentado e fácil de manter.
- Idioma: português do Brasil. Prioridade: celular (uso com uma mão), mas funcional no desktop.
- Escopo da primeira versão: login, biblioteca, tela da cifra, detecção de tom, transposição, ajustes de leitura, exportação em PDF, cadastro e edição de músicas, link da playlist do YouTube e PWA. NÃO fazer agora: setlist, modo apresentação, metrônomo, afinador, tags.
- Referências visuais: as telas da pasta referencias/ (imagens exportadas do protótipo). Siga-as fielmente. O ícone da igreja está em assets/logo-iel.png.

=== 2. IMPORTAÇÃO DOS PDFs (primeira tarefa) ===
Os PDFs estão na pasta documentos/louvor (deixe o caminho configurável no topo do script). Cada PDF representa um dia em que o louvor foi realizado e contém as músicas usadas naquele dia, todas com cifra.
Crie um script Python (pdfplumber ou PyMuPDF) em tools/importar_pdfs.py que:
1. Descubra a data de cada PDF pelo nome do arquivo (aceite 2026-11-16, 16-11-2026, 16.11.26, "Louvor 16-11"). Se não conseguir, use a data de modificação. Mostre uma tabela "arquivo -> data detectada" e peça minha confirmação antes de continuar.
2. Extraia o texto preservando os espaços, porque o alinhamento dos acordes sobre a letra é essencial. Use extração por posição e confira visualmente a primeira página de cada PDF.
3. Separe as músicas dentro de cada PDF, identifique título e tom (se estiver escrito) e distinga linhas de acordes de linhas de letra (uma linha é de acordes quando quase todos os tokens são acordes válidos: C, D#m, Bb7, F#m7(b5), G/B, Asus4, Cadd9 etc.).
4. Se algum PDF for imagem (escaneado), avise e use OCR (pytesseract), com aviso de que a revisão manual é necessária.
5. Remova duplicidades: o mesmo título normalizado (minúsculas, sem acentos, sem pontuação, sem sufixos como "(ao vivo)" ou "- cifra") é a mesma música. Para títulos parecidos (similaridade >= 90%), compare também as primeiras linhas da letra antes de agrupar.
6. REGRA DA VERSÃO: quando a mesma música aparecer em vários PDFs, vale SEMPRE a versão do PDF com a data mais recente. As versões antigas são descartadas, mas registre as datas de uso no campo usedAt (lista) e a última em lastUsedAt.
7. Gere relatorio_importacao.md com: total de músicas únicas, músicas repetidas e suas datas, a versão escolhida e um resumo das diferenças (linhas de letra e acordes que mudaram entre a versão antiga e a nova). Liste também as extrações duvidosas (título não detectado, acordes sem letra, tom ausente).
8. Gere songs.json ordenado alfabeticamente (sem acentos), pronto para importar no Firestore. Nunca versione os PDFs: crie .gitignore com a pasta de PDFs e o JSON, pois letras e cifras têm direitos autorais e o acesso deve ser restrito à equipe.

=== 3. DADOS (Firestore) ===
- songs/{id}: title, titleNorm (sem acento, minúsculo), chart (texto com acordes acima da letra), originalKey, detectedKey, lastUsedAt, usedAt[], createdBy, createdAt, updatedBy, updatedAt.
- settings/app: youtubePlaylistUrl.
- users/{uid}: email, name, role ("admin" | "editor" | "reader"), favorites[], prefs{} (ajustes de leitura e de exportação), keyBySong{songId: tom escolhido}.
- Favoritos, ajustes e tom preferido são por pessoa e não alteram a música original.

=== 4. ACESSO E SEGURANÇA ===
- Login com Google. Só entram e-mails previamente cadastrados pelo admin (allowlist).
- Admin: tudo, mais gerenciar usuários e trocar o link da playlist. Editor: cadastrar, editar e excluir músicas. Leitor: consultar, favoritar, transpor, ajustar a leitura e exportar PDF.
- Escreva as Firestore Security Rules aplicando esses papéis e me ensine a publicá-las e testá-las.

=== 5. FUNCIONALIDADES ===
Biblioteca: ordem alfabética sem acento; busca instantânea por título e por trecho da letra (índice em memória), sem diferenciar acentos; índice A-Z lateral; filtros Todas, Favoritas e Recentes (por lastUsedAt); contador de músicas; virtual scrolling para listas grandes; botão voltar ao topo; Ctrl+F foca a busca no desktop.

Detecção de tom: extraia os acordes, normalize para a raiz (ignore extensões e baixos), pontue cada tom maior e menor pelos acordes diatônicos presentes, dê peso extra ao primeiro e ao último acorde e escolha o de maior pontuação. Em empate (ex.: relativa menor/maior), mostre a sugestão para o editor confirmar. Salve em detectedKey.

Transposição: botões -1 e +1 semitom e seletor direto dos 12 tons. Trate #, b, menores, 7, maj7, m7, sus, add, dim, aug e acordes com baixo (C/E), e escolha a grafia correta conforme o tom (F# ou Gb). Mostre a sugestão de capotraste ("Capo na 2ª casa: toque G para soar em A"). Salve o tom escolhido por pessoa. Inclua testes unitários da transposição (ao menos 30 casos).

Ajustes de leitura (por pessoa, salvos no Firestore e aplicados a todas as músicas):
- Tamanho da fonte A- e A+ (14 a 28 px): acordes e letra escalam juntos em fonte monoespaçada, mantendo o alinhamento, sem alterar o texto salvo.
- Rolagem automática com iniciar/pausar, velocidade de 1 a 10 e toque duplo para pausar.
- Exibir: acordes e letra, só letra ou só acordes.
- Notação: C D E ou Dó Ré Mi.
- Simplificar cifra: reduzir para acordes básicos (sem sétimas, extensões e baixos invertidos).
- Manter tela ligada (Screen Wake Lock API), com aviso se o navegador não suportar.
- Tema: claro, escuro e modo palco. Duas colunas em telas largas. Botão "Restaurar".

Cadastro: tela "Nova música" (título e letra com cifras), aviso de duplicata por título normalizado, tom detectado automaticamente, editar e excluir (admin e editor), atualização em tempo real para toda a equipe (onSnapshot).

Playlist: card "Playlist da equipe" no Início, com botão "Abrir no YouTube", e atalho na Biblioteca. Apenas abre o link (nova aba ou app do YouTube), sem player incorporado. Só o admin altera o link.

Ações da música: copiar, imprimir e exportar PDF.
- Exportar PDF gera um arquivo .pdf de verdade dentro do app, sem depender da tela de impressão do navegador. Use a biblioteca jsPDF (licença MIT) salva localmente na pasta do projeto, sem CDN, para funcionar offline.
- Fonte monoespaçada (Courier embutida do jsPDF), com acentos do português funcionando e acordes alinhados sobre a letra.
- Opções antes de gerar: tom (atual ou original), conteúdo (acordes e letra, ou só letra), tamanho da fonte (10 a 20 pt), mostrar capotraste no topo e "caber em uma página" (reduz a fonte automaticamente). Lembrar as últimas escolhas em prefs.
- Folha A4, título da música no topo e tom indicado. Se a cifra passar de uma página, quebrar entre linhas sem separar um acorde da letra que está logo abaixo.
- Nome do arquivo: "Título - Tom X.pdf".
- O PDF usa o tom escolhido pela pessoa (transposto), nunca altera a música salva.
- Se a geração falhar ou o navegador não permitir o download (comum em alguns iPhones), oferecer como alternativa o botão Imprimir, usando window.print com CSS de impressão (sem menus, fonte monoespaçada, A4, quebra de página sem cortar acordes).
- Imprimir continua existindo como ação separada.

=== 6. OFFLINE E PWA ===
Persistência offline do Firestore e service worker que guarde os arquivos do app, a biblioteca jsPDF e as fontes (inclua as fontes localmente, sem CDN). Após o primeiro acesso, a biblioteca abre sem internet e a exportação de PDF continua funcionando. Login e botão da playlist exigem internet. Manifest com nome "Biblioteca de Louvor", ícones 192 e 512 (inclusive versão maskable) gerados a partir do ícone da igreja e theme_color #2F7A1A. O arquivo atual do ícone é pequeno (176 px): me avise e peça uma versão maior ou em SVG antes de gerar os ícones finais. Instalável no Android e no iPhone.

=== 7. LAYOUT (seguir fielmente as telas de referencias/) ===
Base: 390 px de largura, mobile-first. Alvos de toque de no mínimo 44 px. Ações principais na parte de baixo.

Identidade: o ícone da igreja (círculo verde com cruz) aparece no topo do Início, na Biblioteca (menor) e na tela de login (grande).

Paleta (derivada do verde do ícone, #409722):
- Claro: fundo #FBFAF6; texto #1F2A17; secundário #6B7563; verde escuro #2F7A1A (botões e textos); verde #409722 (somente o ícone); destaque claro #6DBE45 (preenchimentos com texto escuro #14210A); tinta suave #ECF6E6; superfícies #F1F6EE; linhas #EEF0E6.
- Escuro e palco: fundo #0F160A; cards #1C2714; texto #EAF1DF; secundário #A9B79A; destaque #7FD35A.

Tipografia: Fraunces 600 nos títulos (32 a 34 px), Inter 400/600/700 no texto (14 a 16 px), JetBrains Mono 500/700 nas cifras (15 px padrão). Rótulos pequenos em caixa alta, 12 px, espaçamento 0.14em, verde escuro.

Componentes: cards brancos com raio 24 px e sombra suave (0 1px 2px e 0 8px 24px, preto a 4 e 6 por cento); botões em pílula de 52 px; chips de 44 px; linhas de lista de 68 px com selo quadrado de 44 px (raio 14, fundo #ECF6E6) mostrando o tom; ícones de traço de 2 px; transições de 150 a 250 ms.

Barra inferior (88 px, fundo branco): Início, Biblioteca e Nova música (esta só para admin e editor). Item ativo em verde escuro.

Telas:
1) Entrar (referencias/01-entrar.png): ícone da igreja grande (112 px), rótulo "Igreja Evangélica Livre", título "Biblioteca de Louvor", frase "Cifras e repertório da nossa equipe, sempre à mão.", card com o versículo João 8:36 (opcional, fácil de remover), botão "Continuar com Google" e aviso "Acesso restrito à equipe. Use o e-mail cadastrado."
2) Início (referencias/02-inicio.png): ícone de 44 px com o rótulo "Igreja Evangélica Livre", título "Olá, bem-vindo" (saudação neutra, sem expressões religiosas), campo de busca em pílula, card verde escuro (#2F7A1A, raio 28) "Continuar de onde parou" com título da última música aberta, tom e botão "Abrir cifra" (fundo #6DBE45), card "Playlist da equipe" com botão "Abrir no YouTube" e lista "Recentes" com selo de tom e coração.
3) Biblioteca (referencias/03-biblioteca.png): ícone de 28 px com o rótulo "Repertório", título "Biblioteca", contador "N músicas", busca, chips Todas/Favoritas/Recentes, lista agrupada por letra com selo de tom e coração, e índice A-Z vertical à direita.
4) Cifra (referencias/04-cifra.png): seta voltar e coração no topo, título grande, "Tom original G", card branco com o rótulo "TOM", sugestão de capotraste, botões - e + em círculos tintados e o tom atual em Fraunces 30 px; abaixo, a cifra em fonte mono; barra flutuante escura (#1F2A17, raio 30, 60 px) com Copiar, Imprimir, PDF e Aa (Aa abre Ajustes e PDF abre a tela 9).
5) Escolher tom (bottom sheet, referencias/05-escolher-tom.png com fundo escurecido): "Escolha o tom", grade 4x3 com os 12 tons (selecionado em verde escuro), caixa tintada com o capotraste, botão "Aplicar tom X" e "Voltar ao original".
6) Ajustes de leitura (bottom sheet, referencias/06-ajustes-leitura.png): seções TAMANHO DA CIFRA (A-, prévia, A+), ROLAGEM AUTOMÁTICA (Iniciar, velocidade - e +), EXIBIR e NOTAÇÃO (controles segmentados), chaves "Simplificar cifra" e "Manter tela ligada" e botão "Restaurar".
7) Modo palco (escuro) (referencias/07-modo-palco.png): cifra a 18 px ou mais, controles - e + do tom no topo, selo "Modo palco" e rolagem automática em destaque.
8) Nova música (referencias/08-nova-musica.png): campos Título e Letra com cifras (textarea mono de 300 px), aviso de que o tom é detectado automaticamente, botão fixo "Salvar na biblioteca" e ação "Cancelar".
9) Exportar PDF (bottom sheet, referencias/09-exportar-pdf.png com fundo escurecido): título "Exportar PDF" e subtítulo com o nome da música e "folha A4"; controle segmentado TOM (Atual / Original); controle segmentado CONTEÚDO (Acordes e letra / Só letra); TAMANHO DA FONTE com A-, valor em pt e A+; chaves "Mostrar capotraste" (ligada) e "Caber em uma página" (desligada); botão "Gerar PDF" em pílula verde escura de 56 px e "Cancelar".

Acessibilidade: contraste mínimo de 4.5:1 no texto, aria-label em todos os botões de ícone, foco visível, respeitar prefers-reduced-motion e prefers-color-scheme. Nunca usar o verde do ícone (#409722) como cor de texto pequeno sobre branco.

=== 8. COMO TRABALHAR ===
Entregue em etapas e só avance quando eu confirmar:
(1) Importação dos PDFs e relatório. (2) Projeto Firebase, login e regras de segurança. (3) Biblioteca, busca e favoritos. (4) Tela da cifra: tom, transposição, ajustes de leitura e exportação em PDF. (5) Cadastro e edição com permissões. (6) Playlist. (7) PWA, offline e publicação no Firebase Hosting.
Em cada etapa: mostre os arquivos completos, explique como testar e liste riscos. Se algo estiver ambíguo, pergunte em vez de supor.
