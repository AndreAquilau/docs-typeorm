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

#### Configurando ormconfig.js
~~~ts
module.exports = {
  type: process.env.TYPEORM_CONNECTION,
  url: process.env.DATABASE_URL,
  schema: process.env.TYPEORM_SCHEMA,
  synchronize: process.env.TYPEORM_SYNCHRONIZE,
  logging: process.env.TYPEORM_LOGGING,
  entities: [process.env.TYPEORM_ENTITIES || 'dist/models/**/*.js'],
  migrations: [
    process.env.TYPEORM_MIGRATIONS || 'dist/database/migrations/**/*.js',
  ],
  subscribers: [process.env.TYPEORM_SUBSCRIBERS || 'dist/subscriber/**/*.js'],
  cli: {
    entitiesDir: process.env.TYPEORM_ENTITIES_DIR,
    migrationsDir: process.env.TYPEORM_MIGRATIONS_DIR,
    subscribersDir: process.env.TYPEORM_SUBSCRIBERS_DIR,
  },
};
~~~

#### Carregando Variavel de ambiente do typeorm
~~~
...
  |-- src
    |-- config
      |-- env.ts
~~~
~~~ts
import { config } from 'dotenv';
import { resolve } from 'path';

config({
  path: resolve(__dirname, '..', '..', '.env'),
});
~~~

#### Criando conexão com o bando de dados e exportando a conexão
~~~
...
  |-- src
    |-- database
      |-- index.ts
~~~
~~~ts
import 'reflect-metadata';
import { createConnection } from 'typeorm';

export default (async () => {
  return createConnection();
})();
~~~

#### Script typeorm no package.json
~~~json
"scripts": {
    "typeorm": "ts-node-dev -r tsconfig-paths/register ./node_modules/typeorm/cli.js --config ./ormconfig.js ",
}
~~~

### Typeorm CLI
~~~bash
--rodar migrations
yarn typeorm migration:run

--criar uma migration
yarn typeorm migration:create -- -n createUser

--gerar migrations a partir das model
yarn typeorm migration:generate -- -n createUser

--reverte uma magration
yarn typeorm migration:revert


--criar uma model
yarn typeorm entity:create -- -n User
~~~

#### Criando Entity
~~~bash
--criar uma model

yarn typeorm entity:create -- -n User
~~~
~~~ts
import {
  Index,
  Entity,
  Column,
  PrimaryGeneratedColumn,
} from 'typeorm';

@Index('pkey_id_produto', ['id'], { unique: true })
@Entity('produtos', { schema: 'public' })
export class User {
    @PrimaryGeneratedColumn('uuid')
    id: number;

    @Column({
        name: 'nome',
        type: 'varchar',
        length: 255,
        nullable: false
    })
    nome: string;

    @Column({
        name: 'sobrenome',
        type: 'varchar',
        length: 255,
        nullable: false
    })
    sobrenome: string;
    
    @Column({
        name: 'idade',
        type: 'integer',
        nullable: false
    })
    idade: number;

    @CreateDateColumn({
        name: 'created_At',
        type: 'timestamp',
        default: 'now()'
    })
    created_At: Date;

    @UpdateDateColumn({
        name: 'updated_At',
        type: 'timestamp',
        default: 'now()'
    })
    upated_date: Date;

}
~~~