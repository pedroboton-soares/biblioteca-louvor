# Biblioteca de Louvor

App online e instalável (PWA) da Igreja Evangélica Livre para a equipe de louvor: cifras, detecção de tom, transposição, exportação em PDF e cadastro colaborativo de músicas.

## Status
Fase de design concluída. O desenvolvimento ainda não começou. O passo a passo está em [`PROMPT.md`](PROMPT.md).

## Decisões
- **Backend:** Firebase (Authentication com Google, Firestore, Hosting).
- **Front-end:** HTML, CSS e JavaScript puros, sem framework e sem etapa de build.
- **PWA:** abre offline depois do primeiro acesso.
- **Acesso:** login Google, só e-mails autorizados. Papéis: admin, editor e leitor.
- **Músicas:** cada PDF em `documentos/louvor` representa um dia de louvor. Sem repetidas, vale sempre a versão do PDF mais recente.
- **Escopo da v1:** biblioteca, busca, favoritos, cifra com tom e transposição, ajustes de leitura, exportar PDF, cadastro e edição, link da playlist do YouTube.
- **Fora da v1:** setlist, modo apresentação, metrônomo, afinador, tags.

## Estrutura
- `PROMPT.md`: prompt completo para desenvolver o app, em etapas.
- `assets/logo-iel.png`: ícone da igreja (176 px; pedir versão maior ou SVG para os ícones do PWA).
- `prototipo/`: telas do protótipo (código-fonte do canvas de design).
  - Opção A (escolhida): `Entrar`, `Main` (Início), `Biblioteca`, `Cifra`, `Tom`, `Escuro` (modo palco), `Ajustes`, `Exportar`, `Nova`.
  - Opção B (descartada, guardada para consulta): arquivos `B1` a `B7`.
  - Os arquivos `.dc.html` são fonte do canvas e não abrem sozinhos no navegador. Para guardar as telas como imagem, exporte-as do canvas e salve em `referencias/`.
- `referencias/`: imagens exportadas das telas para guiar o desenvolvimento.

## Paleta (derivada do ícone)
Verde escuro `#2F7A1A` (botões e textos), destaque `#6DBE45`, fundo `#FBFAF6`, texto `#1F2A17`. Modo escuro: fundo `#0F160A`, destaque `#7FD35A`. Fontes: Fraunces, Inter e JetBrains Mono.

## Direitos autorais
Letras e cifras são protegidas. Este repositório deve ficar **privado** e nunca deve conter PDFs, `songs.json` nem o relatório de importação (já bloqueados no `.gitignore`). O acesso ao app é restrito à equipe.
