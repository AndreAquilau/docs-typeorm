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
  UpdateDateColumn,
  CreateDateColumn
} from 'typeorm';

@Index('pkey_id_user', ['id'], { unique: true })
@Entity('user', { schema: 'public' })
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

#### Criando migration apartir de uma model
~~~bash
--Gerar migrations a partir das model.

yarn typeorm migration:generate -- -n createUser

--Depois só executar a migration pra tabela ser criada.

yarn typeorm migration:run
~~~

#### Usando getRepository executar processos SGBD 
##### save().
O método save já cria e salva diréto na base de dados. 
~~~ts
import {getRepository} from 'typeorm';
import User from '@model/User';

const repository = getRepository(User);

const user = new User();
user.nome = "Lucas";
user.sobrenome = "Silva";
user.idade = 25;
await repository.save(user);
~~~
##### create() and save().
O método create cria uma instância da model, porém não salva na base de dados.

É muito usado quando quer que seja feito alguma validação na model.
~~~ts
import {getRepository} from 'typeorm';
import User from '@model/User';

const repository = getRepository(User);
//cria um usuário usando a model mas ainda não salva na base de dados.
const user = repository.create({
    nome: 'Lucas',
    sobrenome: 'Silva',
    idade: 25,
})
//salvando usuário na base de dados. já instânciado.
await repository.save(user);
~~~
##### find() and findOne().
O método find retorna todos os registro da tabela que esta vinculado a model.

O método findOne retorna apenas um registro da tabela que esta vinculado a model.
~~~ts
const repository = getRepository(User);

const allUsers = await repository.find(); // retorna todos os registros
const user = await repository.findOne({ sobrenome: "Silva", nome: "Lucas" });
~~~
##### remove().
O método remove remove um ou mais registro retornado de uma consulta do tipo find ou findOne.
~~~ts
// deletando usuário
const user = await repository.findOne({ sobrenome: "Silva", nome: "Lucas" });

await repository.remove(user);
~~~

#### Relacionamento
#### Criando relacinameto one-to-one / one-to-one
~~~ts
import Lesson from './Lesson';

@Entity('content')
export default class Content {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @OneToOne((type) => Lesson, (content) => Content) //eslint-disable-line
  @JoinColumn([{ name: 'fk_id_lesson', referencedColumnName: 'id' }])
  lesson: Lesson;
}


//////////////////////////////////////////////////////////////////////////
import Content from './Content';

@Entity('lesson')
export default class Lesson {
  @PrimaryGeneratedColumn('uuid')
  id: number;

  @OneToOne((type) => Content, (lesson) => Lesson) //eslint-disable-line
  @JoinColumn([{ name: 'fk_id_content', referencedColumnName: 'id' }])
  content: Content;

}
~~~

#### Eager in many-to-one / one-to-many.
Quando definimos o "eager" com "true" falos que quando for feito uma consulta irá ser retornado
daquela model todos os dados da relação entre o relacionamento.
~~~
@ManyToOne((type) => Usuario, (tarefas) => Tarefa, { eager: true })
~~~

#### Criando relacionamento many-to-one / one-to-many
~~~ts
import Usuario from './Usuario';

@Entity('tarefa')
export default class Tarefa {
  @PrimaryGeneratedColumn('uuid')
  private id: string;

  @ManyToOne((type) => Usuario, (tarefas) => Tarefa, { eager: true })
  @JoinColumn([{ name: 'fk_id_usuario', referencedColumnName: 'id' }])
  usuario: Usuario;

}

////////////////////////////////////////////////////////////////

import Tarefa from './Tarefa';

@Entity('usuario')
export default class Usuario {
  @PrimaryGeneratedColumn('uuid')
  private id: string;

  @OneToMany((type) => Tarefa, (usuario) => Usuario)
  @JoinColumn([{ name: 'fk_id_tarefas', referencedColumnName: 'id' }])
  tarefas: Tarefa[];

}
~~~

#### Criando relacionamento many-to-many
Em uma relação de muito para muito só e necessário criar o relacionamento em apenas unas das models
~~~ts
    @ManyToMany(type => Category)
    @JoinTable()
    categories: Category[];
~~~

#### AfterLoad campo virtual gerados após o carregamento da model
~~~ts
@Entity('photos')
export default class Photos {
  url: string;

  @AfterLoad()
  getUrl() {
    return (this.url = `${process.env.BASE_URL}:${process.env.PORT}/${process.env.FILES_STATICS_IMAGES}/${this.filename}`);
  }
}
~~~

#### BeforeInsert método executado antes de salva no banco de dados
~~~ts
@Entity('users')
export default class User {

  @Column({ name: 'password', nullable: false, type: 'varchar', length: 255 })
  @IsEmpty({ message: 'Senha não pode ser vazio' })
  password_hash: string;

  password: string | null;

  @BeforeInsert()
  async encryptPassword() {
    console.log(this);
    this.password_hash = await bcrypt.hash(this.password, 8);
    this.password = null;
  }
}
~~~


