# Aula prática: ransomware e keylogger simulados em Python

Este material apresenta, em formato de passo a passo, um módulo de segurança da informação conduzido por um professor. O foco é entender a estrutura dos exercícios, acompanhar uma demonstração reversível com Fernet e analisar os sinais de risco associados a um keylogger.

> **Escopo:** a demonstração de criptografia usa conteúdo fictício e caminhos de laboratório definidos previamente. Este README não ensina captura global de teclas, ocultação da execução nem envio de registros para contas de e-mail.

## Objetivos da aula

Ao concluir o módulo, a turma deverá saber:

- explicar o que a chave Fernet faz e por que sua proteção é importante;
- descrever geração da chave, cifragem e recuperação de uma amostra;
- reconhecer componentes associados à captura de teclas, execução sem janela e envio de dados;
- relacionar esses comportamentos a medidas de prevenção, detecção e recuperação.

## Preparação conduzida pelo professor

**Professor:** “Antes de abrir o editor, vamos definir o escopo. Usaremos uma VM descartável, sem conexão externa, e uma pasta chamada laboratorio com um arquivo de texto criado só para esta aula. O original e a chave ficam disponíveis durante todo o exercício.”

1. Inicie a VM de treinamento e restaure um snapshot limpo.
2. Desative a rede externa, as pastas compartilhadas e a área de transferência compartilhada.
3. Crie laboratorio/amostra.txt com uma frase fictícia, sem dados pessoais.
4. Guarde uma cópia intacta do arquivo para comparação.
5. Confirme que nenhuma chave, registro ou dado da aula será enviado a um repositório.

---

## Exercício 1 — Ransomware simulado com Fernet

### Etapa 1: explicar a biblioteca e a chave

**Professor:** “A biblioteca cryptography.fernet oferece a classe Fernet, que cifra e valida os dados usando uma chave. Primeiro vamos trabalhar com uma mensagem em memória; depois discutiremos como o mesmo fluxo seria organizado para uma única cópia de teste.”

A linha de importação usada na aula é:

    from cryptography.fernet import Fernet

Para gerar uma chave de demonstração, o script chama Fernet.generate_key(). A chave deve ser guardada localmente, fora do repositório e separada dos arquivos de exemplo. O professor ressalta que publicar a chave junto com os dados cifrados elimina a proteção que a demonstração pretende ilustrar.

Um trecho didático para gerar e guardar uma chave exclusiva do laboratório é:

    from pathlib import Path
    from cryptography.fernet import Fernet

    pasta_lab = Path("laboratorio")
    pasta_lab.mkdir(exist_ok=True)

    caminho_chave = pasta_lab / "chave_demo.key"
    if caminho_chave.exists():
        raise SystemExit("A chave de demonstração já existe; não a substitua.")

    chave = Fernet.generate_key()
    caminho_chave.write_bytes(chave)
    print("Chave de demonstração criada na pasta do laboratório.")

**Professor:** “Observem a verificação antes da gravação. Se executarmos o gerador novamente e substituirmos a chave, poderemos perder a capacidade de recuperar uma amostra cifrada com a chave anterior.”

### Etapa 2: demonstrar cifragem e decifragem em memória

Antes de tocar em arquivos, use uma mensagem inventada. Assim a turma vê a sequência sem alterar o sistema de arquivos:

    chave = Fernet.generate_key()
    cifrador = Fernet(chave)

    mensagem = b"conteudo ficticio para a aula"
    mensagem_cifrada = cifrador.encrypt(mensagem)
    mensagem_recuperada = cifrador.decrypt(mensagem_cifrada)

    print("Cifrada:", mensagem_cifrada)
    print("Recuperada:", mensagem_recuperada.decode("utf-8"))

O professor percorre cada linha:

1. Fernet.generate_key() cria uma chave para a demonstração.
2. Fernet(chave) cria o objeto que fará as operações.
3. encrypt() recebe bytes e retorna o conteúdo cifrado.
4. decrypt() valida e recupera o conteúdo usando a mesma chave.
5. A conversão para texto ocorre somente para exibir a mensagem fictícia.

### Etapa 3: planejar os scripts de arquivo sem sobrescrever a amostra

Depois da etapa em memória, a turma desenha a lógica dos dois scripts de arquivo. Os caminhos são fixos para manter a atividade limitada à amostra:

| Arquivo | Papel na demonstração |
|---|---|
| laboratorio/amostra.txt | Original sintético; deve permanecer intacto. |
| laboratorio/amostra.cifrada | Nova saída da demonstração de cifragem. |
| laboratorio/amostra.recuperada.txt | Nova saída da demonstração de recuperação. |
| laboratorio/chave_demo.key | Chave local; não deve ser publicada. |

**Script de cifragem — sequência explicada pelo professor:**

1. Importar Path e Fernet.
2. Definir explicitamente o caminho da única amostra autorizada.
3. Confirmar que a amostra existe e que o caminho de saída não contém outro arquivo.
4. Ler a amostra como bytes e carregar a chave criada para esta aula.
5. Criar um objeto Fernet com essa chave.
6. Cifrar os bytes e gravar o resultado em amostra.cifrada.
7. Exibir uma mensagem de conclusão que informe que o original foi preservado.

**Script de recuperação — sequência explicada pelo professor:**

1. Definir o caminho fixo de amostra.cifrada e o destino amostra.recuperada.txt.
2. Ler a chave local e criar o objeto Fernet.
3. Ler e decifrar somente a amostra de laboratório.
4. Gravar o texto recuperado em um arquivo novo, sem substituir o original.
5. Comparar a amostra inicial e a recuperada para confirmar a restauração.

**Professor:** “A diferença entre esta demonstração e um comportamento destrutivo está no escopo e na reversibilidade: um arquivo sintético, caminhos definidos, saídas separadas, chave preservada e nenhuma busca por outros arquivos.”

### Etapa 4: simular a mensagem de resgate sem extorsão

O curso pode criar um arquivo aviso_simulacao.txt para representar o impacto visual, sem cobrança ou ameaça:

> **SIMULAÇÃO DIDÁTICA — nenhum resgate real.** Uma cópia de um arquivo sintético foi cifrada para a aula. O original e a chave permanecem no laboratório. A demonstração pode ser revertida com a cópia de segurança.

O professor explica que uma mensagem de incidente serve para discutir comunicação e resposta; não é necessário pedir pagamento nem esconder a chave.

### Etapa 5: executar e encerrar a demonstração

A turma executa os scripts preparados pelo instrutor nesta ordem: gerador de chave, cifragem da amostra e recuperação. O resultado esperado é ver o arquivo cifrado aparecer, manter amostra.txt legível e recuperar uma cópia cujo conteúdo corresponde ao original. Ao final, a turma remove os artefatos temporários ou restaura o snapshot.

O exercício não inclui busca por diretórios, cifragem em massa, substituição destrutiva, exclusão de backups, persistência ou propagação.

---

## Exercício 2 — Análise de um keylogger

### Etapa 1: identificar o papel dos componentes estudados

**Professor:** “Agora vamos analisar por que certas combinações de bibliotecas podem transformar um programa comum em uma ameaça. Vamos identificar os papéis no código e os riscos, sem montar ou executar um capturador global.”

| Componente citado no curso | Papel que o professor explica | Risco a reconhecer |
|---|---|---|
| pynput.keyboard | Pode observar eventos de teclado em escopo amplo. | Pode registrar o que é digitado fora de uma aplicação de teste, incluindo credenciais. |
| Arquivo .txt | Pode guardar registros de entrada. | O arquivo passa a conter dados sensíveis e precisa ser protegido e removido. |
| threading.Timer | Agenda uma ação para ocorrer posteriormente ou após um intervalo. | Uma tarefa agendada pode continuar sem interação evidente do usuário. |
| Extensão .pyw | No Windows, inicia Python sem a janela de console habitual. | A execução fica menos visível; o aluno deve reconhecer isso como sinal de ocultação. |
| smtplib | Componente de comunicação com um servidor de e-mail. | Pode permitir saída de dados da máquina. |
| email.mime.text.MIMEText | Formata o corpo textual de uma mensagem. | Pode transportar conteúdo coletado para outro destino. |
| Senha de aplicativo | Credencial que pode autorizar acesso a uma conta de e-mail. | Se for exposta em código ou repositório, pode comprometer a conta. |

### Etapa 2: acompanhar o fluxo em uma amostra fornecida pelo instrutor

O professor apresenta o fluxo somente para análise de defesa, sem fornecer um script operacional:

1. **Origem dos dados:** localizar no código qual componente recebe eventos de teclado.
2. **Registro local:** identificar se os eventos são agregados ou gravados em um arquivo.
3. **Agendamento:** verificar se há chamadas temporizadas que repetem uma ação.
4. **Visibilidade:** observar se o programa informa ao usuário que está ativo ou se a execução fica escondida.
5. **Comunicação externa:** identificar se há conexão de e-mail ou outro canal de saída.
6. **Segredos:** verificar se há credenciais embutidas ou armazenadas de forma insegura.
7. **Resposta:** anotar como a equipe poderia interromper a atividade, preservar evidências e proteger a conta envolvida.

A análise deve ser feita em código de treinamento, sem executar captura de teclas reais. O objetivo é reconhecer a cadeia de risco e explicar como interrompê-la, não reproduzi-la.

### Etapa 3: discutir a etapa de e-mail e a senha de aplicativo

O professor descreve em nível conceitual a relação entre smtplib, MIMEText e um serviço de e-mail: um componente prepara a mensagem e outro pode transmiti-la. Uma senha de aplicativo é uma credencial de acesso e não deve ser colocada em scripts, arquivos de configuração versionados, mensagens ou repositórios públicos.

Este material não contém código de envio de registros, configuração de Gmail, instruções para criar senha de aplicativo ou passos para ocultar o processo. Se uma credencial real já tiver sido criada para a aula, remova-a do material e revogue-a na conta correspondente.

---

## Reflexões sobre defesa

| Controle | O que o professor pede que a turma verifique |
|---|---|
| Antivírus e EDR | Proteção atualizada, alertas de alterações atípicas em muitos arquivos e programas incomuns que observem entrada do usuário. |
| Firewall | Regras de saída compatíveis com a função do dispositivo e investigação de tráfego inesperado. |
| Sandboxing | VM descartável, isolada, sem compartilhamentos e restaurada ao fim. Sandbox reduz risco, mas não substitui supervisão. |
| Cópias de segurança | Cópias offline ou imutáveis e restaurações testadas regularmente. |
| Menor privilégio | Scripts e usuários sem privilégios administrativos desnecessários. |
| Proteção de credenciais | Senhas únicas, MFA e nenhum segredo armazenado em código ou enviado para repositório. |
| Conscientização | Reconhecimento de anexos e links suspeitos e comunicação rápida de comportamento anormal. |
| Resposta a incidentes | Isolar o dispositivo, avisar a equipe responsável, preservar evidências e recuperar dados de cópias verificadas. |

## Fechamento da aula

O objetivo é compreender o comportamento para defender sistemas. A demonstração Fernet ensina cifragem, gestão de chaves e recuperação. A análise do keylogger ensina a reconhecer captura de entrada, ocultação e saída de dados, sem ensinar sua implementação ou operação.

