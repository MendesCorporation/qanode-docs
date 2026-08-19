# Instalação


---

## Versão Desktop 

A versão desktop é a forma mais fácil de começar. Ela inclui todos os componentes necessários em um único instalador, incluindo um banco de dados PostgreSQL embutido.

### Requisitos

- **Windows** 10/11, 64 bits
- **macOS** 11 ou superior, Intel ou Apple Silicon
- **Linux** 64 bits com suporte a AppImage
- **Memória RAM**: 8 GB mínimo (16 GB recomendado)
- **Espaço em Disco**: 2 GB para instalação, além do espaço usado pelas evidências e execuções

### Instalação no Windows

1. Baixe o instalador `QANode Setup.exe` no site https://www.qanode.com
2. Execute o instalador e siga as instruções
3. Ao finalizar, o QANode será aberto automaticamente

[Instalador do QANode no Windows](../../assets/images/instalacao-nsis.png)
*Imagem: Tela do instalador NSIS do QANode no Windows*

### Instalação no macOS

1. Baixe o arquivo `.dmg` correspondente ao seu processador: Apple Silicon (`arm64`) ou Intel (`x64`)
2. Abra o DMG e mova o QANode para **Aplicativos**
3. Abra o QANode pela pasta **Aplicativos** ou pela busca do macOS

O aplicativo é assinado e notarizado pela Apple. Após uma atualização, o macOS pode levar alguns instantes para atualizar o ícone e o índice de busca.

### Instalação no Linux

1. Baixe o arquivo `.AppImage`
2. Marque o arquivo como executável
3. Abra o AppImage

Exemplo:

```bash
chmod +x QANode-Setup-x64.AppImage
./QANode-Setup-x64.AppImage
```

Na primeira execução, o QANode irá:
- Inicializar o banco de dados PostgreSQL embutido
- Criar as tabelas necessárias (migrations automáticas)
- Iniciar o servidor da API
- Abrir a interface do aplicativo

> **Nota:** A primeira inicialização pode levar alguns segundos a mais enquanto o banco de dados é preparado. Uma tela de splash será exibida durante esse processo.

### Pasta de Dados

A versão desktop armazena banco, evidências, arquivos e logs na pasta de dados do usuário:

```text
Windows: %APPDATA%\@qanode\desktop\
macOS:   ~/Library/Application Support/@qanode/desktop/
Linux:   ~/.config/@qanode/desktop/
```

Dentro dela, `postgres/` contém o banco embutido, `storage/` guarda os artefatos e `desktop-main.log` registra a inicialização do aplicativo.

> **Importante:** Nunca modifique ou exclua manualmente a pasta `postgres/`. Antes de qualquer procedimento de recuperação, faça uma cópia da pasta de dados completa.

---


## Solução de Problemas

### A versão desktop não inicia

- Verifique se outra instância do QANode não está em execução
- Abra `desktop-main.log` na pasta de dados e procure por `Fatal startup error`
- No Windows, confirme se o Microsoft Visual C++ Redistributable x64 está atualizado
- Não apague a pasta `postgres/`; preserve o banco e envie o log ao suporte para diagnóstico

---

## Próximos Passos

- [Início Rápido](inicio-rapido.md) — Crie seu primeiro teste
- [Conceitos Fundamentais](conceitos.md) — Entenda a estrutura da plataforma
