### Documentação Typeorm

#### Starting Project Typeorm
~~~bash
yarn add typeorm pg reflect-metadata uuidv4 dotenv && yarn add -D @types/dotenv

or

npm install typeorm pg reflect-metadata uuidv4 && npm install -D @types/dotenv
~~~

#### Criar variável de ambiente .env
typeorm.config.env
~~~env

//Qual SGBD está sendo usado na conexão.
TYPEORM_CONNECTION = postgres 

//Em caso da base de dados trabalhar com schema definir qual vai ser usado default "public"
TYPEORM_SCHEMA = public

//URL de conexão com banco de dados
DATABASE_URL = postgres://postgres:root@localhost:5432/database-app

//Opção true define que qualquer alteração nas models irá refletir na base de dados.
TYPEORM_SYNCHRONIZE = false

//definir os loggins de processos executados pelo SGBD.
TYPEORM_LOGGING = false

//Caminho das entidades / models onde seram executadas
TYPEORM_ENTITIES = ./src/models/**/*.ts 

//Caminho das migrations onde seram executadas
TYPEORM_MIGRATIONS = ./src/database/migrations/**/*.ts

//Caminho das subscribers onde seram executadas
TYPEORM_SUBSCRIBERS = ./src/subscribers/**/*.ts

//Caminho onde seram criados as models
TYPEORM_ENTITIES_DIR = ./src/models/

//Caminho onde seram criados as migrations
TYPEORM_MIGRATIONS_DIR = ./src/database/migrations/

//Caminho onde seram criados as subscribers
TYPEORM_SUBSCRIBERS_DIR = ./src/subscribers/

//tipo de codificação
TYPEORM_DRIVER_EXTRA='{"charset": "utf8mb4"}'
~~~