# Criando uma API com FastAPI utilizando Test Driven Development (TDD)

Este guia detalha como criar uma API simples com FastAPI utilizando a metodologia Test Driven Development (TDD).

## Pré-requisitos

*   **Python 3.8+:** Certifique-se de ter uma versão do Python igual ou superior a 3.8 instalada.
*   **Poetry (Opcional):** Recomendado para gerenciamento de dependências. Se não o tiver, pode usar `pip`.
*   **Conhecimento básico de Python:** Familiaridade com a sintaxe e conceitos básicos da linguagem.
*   **Conhecimento básico de FastAPI:**  Entendimento dos princípios básicos de como FastAPI funciona (rotas, modelos Pydantic, etc.).

## Passo a Passo

### 1. Configuração do Projeto

1.  **Criar o diretório do projeto:**

    ```bash
    mkdir fastapi-tdd-example
    cd fastapi-tdd-example
    ```

2.  **Inicializar o projeto (com Poetry):**

    ```bash
    poetry init --name fastapi-tdd-example --description "Exemplo de API com FastAPI e TDD" --author "Seu Nome <seu.email@exemplo.com>" --python "^3.8"
    ```
    Preencha as informações solicitadas.

    **Ou (sem Poetry):**
    Crie um arquivo `requirements.txt` para listar as dependências posteriormente.

3.  **Criar o ambiente virtual (se não estiver usando Poetry):**

    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    ```

4.  **Estrutura de diretórios:**

    ```
    fastapi-tdd-example/
    ├── app/           # Código da API
    │   ├── __init__.py
    │   ├── main.py     # Arquivo principal da API
    │   └── models.py   # Definição de modelos Pydantic
    ├── tests/         # Testes unitários
    │   ├── __init__.py
    │   └── test_main.py # Testes para a API
    ├── poetry.toml     # (Se usar Poetry)
    └── README.md       # Este arquivo
    ```

### 2. Escrevendo o Primeiro Teste (TDD)

1.  **Instalar as dependências (com Poetry):**

    ```bash
    poetry add fastapi uvicorn pytest requests python-multipart
    poetry add -D pytest pytest-cov
    ```

    **Ou (sem Poetry):**

    Adicione ao `requirements.txt`:

    ```
    fastapi
    uvicorn
    pytest
    requests
    python-multipart
    ```

    E instale:

    ```bash
    pip install -r requirements.txt
    ```

    **Explicação das dependências:**
    *   `fastapi`: Framework web para criar a API.
    *   `uvicorn`: Servidor ASGI para rodar a API.
    *   `pytest`: Framework para escrever e executar testes.
    *   `requests`: Para fazer requisições HTTP nos testes.
    *   `python-multipart`: Necessário para tratamento de uploads (se precisar).
    *   `pytest-cov`: Para medir a cobertura de código dos testes.

2.  **Escrever o primeiro teste em `tests/test_main.py`:**

    ```python
    from fastapi.testclient import TestClient
    from app.main import app

    client = TestClient(app)

    def test_read_main():
        response = client.get("/")
        assert response.status_code == 200
        assert response.json() == {"message": "Hello World"}
    ```

    **Explicação:**
    *   Importa `TestClient` do `fastapi.testclient` para simular requisições HTTP na API.
    *   Importa a instância `app` do FastAPI do `app/main.py`.
    *   Cria uma instância do `TestClient` com a aplicação FastAPI.
    *   Define um teste `test_read_main` que faz uma requisição GET para a rota `/`.
    *   Verifica se o status code da resposta é 200 (OK).
    *   Verifica se o corpo da resposta é um JSON contendo `{"message": "Hello World"}`.

### 3. Executar o Teste (Falhará)

1.  **Executar o teste:**

    ```bash
    pytest
    ```

    Você verá um erro, pois a API ainda não foi implementada. O teste falhará indicando `ModuleNotFoundError: No module named 'app'`.

### 4. Implementar o Código Mínimo para Passar no Teste

1.  **Criar o arquivo `app/main.py` com o seguinte conteúdo:**

    ```python
    from fastapi import FastAPI

    app = FastAPI()

    @app.get("/")
    async def read_root():
        return {"message": "Hello World"}
    ```

    **Explicação:**
    *   Importa `FastAPI`.
    *   Cria uma instância do FastAPI chamada `app`.
    *   Define uma rota GET na raiz (`/`) usando o decorador `@app.get("/")`.
    *   A função `read_root` retorna um dicionário que será automaticamente convertido em JSON.

### 5. Executar o Teste (Passará)

1.  **Executar o teste novamente:**

    ```bash
    pytest
    ```

    Desta vez, o teste deve passar.

### 6. Refatorar (Opcional)

Nesta fase, você pode refatorar o código para melhorar a legibilidade, performance ou estrutura, mantendo os testes passando.  Neste caso, o código inicial já está bastante simples, então a refatoração pode ser mínima.

### 7. Iterar: Adicionar Nova Funcionalidade com TDD

Agora, vamos adicionar uma nova funcionalidade: um endpoint `/items/{item_id}` que retorna informações sobre um item.

1.  **Escrever o teste para a nova funcionalidade em `tests/test_main.py`:**

    ```python
    from fastapi.testclient import TestClient
    from app.main import app

    client = TestClient(app)

    def test_read_main():
        response = client.get("/")
        assert response.status_code == 200
        assert response.json() == {"message": "Hello World"}


    def test_read_item():
        response = client.get("/items/123")
        assert response.status_code == 200
        assert response.json() == {"item_id": "123"}
    ```

    **Explicação:**
    *   Adiciona um novo teste `test_read_item` que faz uma requisição GET para a rota `/items/123`.
    *   Verifica se o status code da resposta é 200.
    *   Verifica se o corpo da resposta é um JSON contendo `{"item_id": "123"}`.

2.  **Executar os testes (Falhará):**

    ```bash
    pytest
    ```

    O novo teste falhará, pois a rota `/items/{item_id}` ainda não foi implementada.

3.  **Implementar o código mínimo para passar no teste em `app/main.py`:**

    ```python
    from fastapi import FastAPI

    app = FastAPI()

    @app.get("/")
    async def read_root():
        return {"message": "Hello World"}

    @app.get("/items/{item_id}")
    async def read_item(item_id: str):
        return {"item_id": item_id}
    ```

    **Explicação:**
    *   Adiciona uma nova rota GET em `/items/{item_id}` usando o decorador `@app.get("/items/{item_id}")`.
    *   A função `read_item` recebe o `item_id` como parâmetro (automaticamente extraído da URL pelo FastAPI).
    *   Retorna um dicionário contendo o `item_id`.

4.  **Executar os testes (Passará):**

    ```bash
    pytest
    ```

    Todos os testes devem passar agora.

5.  **Refatorar (Opcional):** Avalie se há necessidade de refatorar.

### 8. Adicionando Validação e Tipagem com Pydantic

Vamos adicionar um modelo Pydantic para validar os dados do item e tipar a resposta.

1.  **Criar o arquivo `app/models.py` com o seguinte conteúdo:**

    ```python
    from pydantic import BaseModel

    class Item(BaseModel):
        name: str
        description: str | None = None
        price: float
        tax: float | None = None
    ```

    **Explicação:**
    *   Importa `BaseModel` do `pydantic`.
    *   Define um modelo `Item` que herda de `BaseModel`.
    *   Define os campos `name` (string obrigatória), `description` (string opcional), `price` (float obrigatório) e `tax` (float opcional).

2.  **Escrever o teste para criar um item em `tests/test_main.py`:**

    ```python
    from fastapi.testclient import TestClient
    from app.main import app

    client = TestClient(app)

    def test_read_main():
        response = client.get("/")
        assert response.status_code == 200
        assert response.json() == {"message": "Hello World"}


    def test_read_item():
        response = client.get("/items/123")
        assert response.status_code == 200
        assert response.json() == {"item_id": "123"}


    def test_create_item():
        item_data = {
            "name": "Example Item",
            "description": "A description of the item",
            "price": 10.99,
            "tax": 1.20,
        }
        response = client.post("/items/", json=item_data)
        assert response.status_code == 200
        assert response.json() == {
            "name": "Example Item",
            "description": "A description of the item",
            "price": 10.99,
            "tax": 1.20,
        }
    ```

    **Explicação:**
    *   Adiciona um novo teste `test_create_item` que faz uma requisição POST para a rota `/items/`.
    *   Define um dicionário `item_data` com os dados do item.
    *   Envia os dados como JSON no corpo da requisição.
    *   Verifica se o status code da resposta é 200.
    *   Verifica se o corpo da resposta é igual aos dados enviados.

3.  **Executar os testes (Falhará):**

    ```bash
    pytest
    ```

    O novo teste falhará, pois a rota POST `/items/` ainda não foi implementada.

4.  **Implementar o código mínimo para passar no teste em `app/main.py`:**

    ```python
    from fastapi import FastAPI
    from pydantic import BaseModel

    app = FastAPI()

    class Item(BaseModel):
        name: str
        description: str | None = None
        price: float
        tax: float | None = None


    @app.get("/")
    async def read_root():
        return {"message": "Hello World"}

    @app.get("/items/{item_id}")
    async def read_item(item_id: str):
        return {"item_id": item_id}


    @app.post("/items/")
    async def create_item(item: Item):
        return item
    ```

    **Explicação:**
    *   Importa o modelo `Item` de `app/models.py`.
    *   Adiciona uma nova rota POST em `/items/` usando o decorador `@app.post("/items/")`.
    *   A função `create_item` recebe um objeto `Item` como parâmetro.  O FastAPI se encarrega de validar os dados do corpo da requisição com o modelo Pydantic.
    *   Retorna o objeto `Item` recebido.

5.  **Executar os testes (Passará):**

    ```bash
    pytest
    ```

    Todos os testes devem passar agora.

### 9. Coverage de Testes

Para verificar a cobertura dos seus testes, execute o seguinte comando:

```bash
pytest --cov=app
```

Isso mostrará a porcentagem de código coberta pelos testes. O objetivo é ter uma cobertura alta para garantir que seu código está bem testado.  Idealmente, mire em uma cobertura acima de 80%.

### 10. Próximos Passos

*   **Adicionar mais testes:** Escreva testes para todos os cenários possíveis (erros, casos de borda, etc.).
*   **Adicionar validação mais complexa:** Use Pydantic para validar os dados de entrada de forma mais granular.
*   **Integrar com um banco de dados:** Conecte a API a um banco de dados para persistir os dados.
*   **Adicionar autenticação e autorização:** Proteja a API com mecanismos de autenticação e autorização.
*   **Documentar a API:** Use FastAPI para gerar automaticamente a documentação da API com Swagger UI.

## Conclusão

Este guia demonstrou como criar uma API simples com FastAPI usando TDD.  Lembre-se que o ciclo TDD é um processo iterativo:

1.  Escreva um teste (que falha).
2.  Implemente o código mínimo para passar no teste.
3.  Refatore (se necessário).
4.  Repita o processo para adicionar novas funcionalidades.

Seguindo essa metodologia, você pode garantir que seu código está bem testado desde o início, o que facilita a manutenção e evolução da API a longo prazo.