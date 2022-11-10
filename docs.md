Tài liệu bao gồm: 

- [Cấu trúc dự án](#cấu-trúc)

- [Hướng dẫn setting](#hướng-dẫn-setting)

- [Flow cơ bản để truy xuất/ cập nhật dữ liệu](#flow-cơ-bản-để-truy-xuất-cập-nhật-dữ-liệu)

# Cấu trúc
- [_env](#_env)
    - [.env.example]
- data (updating)
- public (updating)
- src
    - [cron_job](#cron_job)
    - db
        - [entity](#entity)
    - event (updating)
    - graphql
        - [schema](#schema)
    - [i18n](#i18n)
    - [interfaces](#interfaces-updating)
    - libs (updating)
    - [middleware](#middleware)
    - [repotitories](#repotitories)
    - [routes](#routes)
    - [services](#services)
    - test (updating)
    - [utils](#utils)
## _env
Thư mục chứa file setting của dự án, có file setting mẫu `.env.example`, các setting hiện tại:
- HOST, PORT, môi trường đang chạy dự án.
- JWT key.
- MICRO Service configs
- Application config
- CORS
- Database config
- Email project

## cron_job
- Setup các chức năng chạy định kỳ trong dự án.

## entity
- Chức các file khai báo các class entity sử dụng bởi `typeORM`

## schema
- Khai báo `schema` định nghĩa các kiểu dư liệu sẽ sử dụng trong `GraphQL`

## i18n
- Thư viện khai báo các message có trong dự án.

## interfaces (updating)
- Khai báo các interface, sử dụng để xác định các type của params, response sử dụng cho `resolvers`

## middleware
- Hiện tại đang chứa các function dùng để kiểm tra quyền truy cập(role), trạng thái login, cors của user

## repotitories
- Nơi khái báo các `repository` cho từng entity, chứ các function có thể thao tác với database

## routes

- Khai báo endpoint người dùng/dev có thể truy cập được không thông qua `GraphQL`


## services

- Khai báo các `service`, các lớp mà `resolvers` của `GraphQL` thông qua nó để sử dụng các `repository` truy cập vào database 

## utils
- Chứa contants, enum, helper mà được sử dụng chung ở toàn dự án.


# Hướng dẫn setting

- B1: Nếu chạy ở máy local thì phải tạo Database, schema trong postgres trước
    - Tên database (trong ví dụ này là work)
    - Các schema cần thiết
        - data
        - org
        - reports
        - system
        
- B2: Tạo file setting môi trường 
    - Tạo file _env/`.env.development` từ file _env/`.env.example`
    - Cập nhật lại dòng NODE_ENV=production thành NODE_ENV=develoment
    - Cập nhật lại DB config (lấy các thông tin của postgres như hình dưới điền vào DB config)
    
    ![image](https://user-images.githubusercontent.com/36842493/201005811-54049a9f-72ea-404e-9956-fda44f238314.png)

 ```
    # DB config
    DB_HOST=localhost
    DB_PORT=5432
    DB_NAME=work
    DB_USERNAME=postgres
    DB_PASSWORD= :)))
    DB_LOGGING=false
```
- B3 làm theo
        
# Flow cơ bản để truy xuất/ cập nhật dữ liệu

Tạo các file cần lượt theo các thư mục dưới đây.

[entity](#entity-httpstypeormioentities) -> [repositories](#repositories) -> [services](#services) -> [graphql](#graphql) -> [Sử dụng](#sử-dụng)

### entity https://typeorm.io/entities
Khởi tạo class entity để ánh xạ một class với một table trong database thông qua `typeORM`

-> Tạo class Car theo path src/db/entity/`car.entity.ts`

    import { Column, Entity, PrimaryGeneratedColumn, UpdateDateColumn } from "typeorm";

    
    @Entity({name: 'car', schema: 'product', synchronize: true})
    export class Car {
            @PrimaryGeneratedColumn()
            id: number

            @Column( { type: 'text', nullable: false})
            name: string

            @Column( { nullable: true } )
            color: string

            @Column({ type: 'decimal', nullable: true})
            price: number

            @Column({ nullable: true })
            origin: string

            @Column({ type: 'int', default: -1 })
            status: number;

            @Column({ type: 'boolean', default: false })
            removed: boolean;

            @Column({ type: 'uuid', nullable: true })
            creator: string;

            @Column({ type: 'uuid', nullable: true })
            updater: string;

            @Column({ type: 'uuid', nullable: true })
            remover: string;

            @CreateDateColumn({ type: 'timestamptz', default: () => 'CURRENT_TIMESTAMP' })
            created_at: Date;

            @UpdateDateColumn({ type: 'timestamptz', default: () => 'CURRENT_TIMESTAMP' })
            updated_at: Date;

            @Column({ type: 'timestamptz', nullable: true })
            removed_at: Date;
    }

* Thông tin thêm: 
    Các decorator @Entity, @Column,... nhằm đánh dấu class, property giúp cho `typeORM` có thể hiểu được class này là một entity dùng để ánh xạ vào database
    - Decorator
      - @Entity đánh dấu class để `typeORM` có thể hiểu và ánh xạ với một table ở database.
        - Một số option của decorator @Entity 
          - name tùy chọn tên table khi ánh xạ vào database
          - schema chọn schema mà class sẽ ánh xạ vào
          - synchronize đồng bộ class entity với table trong database 

            nếu synchronize: true thì mọi thay đổi của class đều sẽ cập nhật vào table trong database.

      - @Column đánh dấu đây là một cột trong table.
        - type tùy chọn type cho các column
        - nullable column này cho phép có trị null hoặc không
      - Các decorator đặc biệt:
        - @PrimaryGeneratedColumn cột khóa chính tự tăng.
        - @CreateDataColumn cột tự lấy giá trị date khi tạo mới một đối tượng.
        - @UpdateDateColumn cột tự lấy giá trị date khi cập nhật một đối tượng.

### repositories

 -> Tạo file src/repositories/`car.repository.ts`

    import { CarEntity } from "src/db/entity/car.entity"
    import RepoAbstract from "./repo.abstract"

    class _CarRepository extends RepoAbstract<CarEntity>{
        constructor(){
            super(CarEntity, [])
        }
        
        // Bên trong class RepoAbstract đã có sẵn một số function dể truy vấn/ cập nhật database, ta có thể kế thừa và sử dụng
        // Hoặc nếu không có function phù hợp ta có thể tạo các function dùng riêng cho CarRepository ở đây.
        
        async isDuplicate(carName: string): Promise<Boolean> {
            const car = await this.getRepo().findOneBy({name: carName});
            return Promise.resolve(!!car);
        }
    }

    export const CarRepository = _CarRepository;

- Thông tin thêm:
    
    - Trong project hiện tại, chúng ta sử dụng Data Mapper pattern, ở đây mọi truy vấn và cập nhật dữ liệu điều thông qua repository.

      File repositories/`repo.abstract.ts` file này export class RepoAbstract là một lớp trừu tượng, class sẽ khởi tạo một repository cho ta để connect đến database.

      Nên để có thể truy vấn hoặc cập nhật dữ liệu chỉ cần tạo một class repository kế thừa class RepoAbstract.
      
    - Một số function có sẵn trong RepoAbstract:
        - getRepo() nếu muốn sử dụng các function của `TypeORM` (findOne, findOneBy,...) thì ở các repository kế thừa từ RepoAbstract phải thông qua function getRepo() 
          
          vd: this.getRepo().finOneBy({name: 'value'});
          
        - create(payload, actor) tạo mới một instance
            - payload là đối tượng tạo mới
            - actor id của người tạo mới đối tượng
        - detail() lấy chi tiết của một đối tượng (lưu ý table phải có column removed, nếu không dùng hàm này sẽ lỗi)
        - listAll() lây danh sách các đối tượng
    
### services

Các service là một lớp nằm giữa graphql (nhận yêu cầu truy vấn/ cập nhật) và repository (có quyền truy cập vào dữ liệu).

Ở đây chúng ta khai báo các class service chứa các service function riêng cho từng entity.

=> Tạo file src/services/car.service.ts

    import { ApolloError } from "apollo-server-express";
    import { Request } from "express";
    import { CarEntity } from "src/db/entity/car.entity";
    import { CarRepository } from "src/repositories/car.repository";
    import { getAcceptLanguageFromHeader } from "src/utils";
    import { DeepPartial } from "typeorm";

    interface ICarService {
        getCarById(id: number): Promise<CarEntity | null>;
        createCar(data: DeepPartial<CarEntity>, request: Request): Promise<Boolean>;
    }

    export const CarService: ICarService = {
        // truy vấn dữ liệu
        async getCarById(idCar) {
            return await CarRepository.detail({id: idCar});
        },
        async getCars(){
            return  await CarRepository.listAll();
        },
               
        // cập nhật dữ liệu
        async createCar(data, request){
            const check = !!data.name && CarRepository.isDuplicate(data.name);
            if(check)
            {
                throw new ApolloError(
                    i18n.__({
                      phrase: 'user.error.username.is_existed',
                      locale: getAcceptLanguageFromHeader(request),
                    }),
                    'BAD_REQUEST'
                  );
            }
            return (!!await CarRepository.create(data, null));
        }
    }
    
  - Thông tin thêm: 
    - ApolloError() function để throw error cho dự án.
    - i18n thư viện để tự tùy chỉnh message error thành tiếng việt/ tiếng anh.
    
 ### graphql
 1. Khai báo `schema` cho graphql: 
 Tạo file src/graphql/schema/car/`car.graphql`
 
 ```
    type Car {
        id: ID
        name: String
        color: String
        price: Int
        origin: String
    }

    input InputCar {
        name: String!
        price: Int!
    }

    type Query {
        get_cars: [Car]
        get_car_by_id(id: Int!): Car
    }

    type Mutation {
        create_car(data: InputCar!): Boolean
    }
        
 ```
 
 - Thông tin thêm `schema` (GraphQL schema) là ngôn ngữ sử dụng trong GraphQL bao gồm các type và field của các type.
 - Một số dạng type của GraphQL schema 
    - Object type (type Car): xác định một đối tượng chứa các field có thể truy vấn trong graphql
    - Query type (type Query): loại đặt biệt của graphql chứa các field xác định cho một yêu cầu truy vấn dữ liệu có thể thực hiện
    - Mutation type (type Mutation):  loại đặt biệt của graphql chứa các field xác định yêu cầu cập nhật dữ liệu có thể thực hiện
    - Scalar type (Int, String, Boolean,...): các type cơ bản trong GraphQL như dùng để xác định type của các field  
    - Input type(input ICar): xác định một đối tượng dùng làm đối số để truyền vào yêu cầu truy vấn/ cập nhật
    - Subscriptions type: (đang cập nhật...)
    
 2. Sau khi khai báo `schema` cho graphql, chúng ta tiếp tục tạo các `resolver` cho từng field của các type
 - Tạo file src/graphql/schema/car/`query.resolvers.ts`
 
        import { combineResolvers } from "graphql-resolvers";
        import { AuthorizationUser } from "src/middleware/auth.middleware";
        import { CarService } from "src/services/car.service";

        export default {
            Query: {
                get_cars: combineResolvers(
                    AuthenticationUser,
                    async () => {
                        return await CarService.getCars();
                    }
                ),
                get_car_by_id: async (_:any, { id }: {id:number}) => {
                    return await CarService.getCarById(id);
                }
            }
        }
        
 - Tạo file src/graphql/schema/car/`mutation.resolvers.ts`
    
        import { Request } from "express";
        import { combineResolvers } from "graphql-resolvers";
        import { CarEntity } from "src/db/entity/car.entity";
        import { AuthorizationUser } from "src/middleware/auth.middleware";
        import { CarService } from "src/services/car.service";
        import { DeepPartial } from "typeorm";

        export default {
            Mutation: {
                create_car: combineResolvers(
                    AuthorizationUser(['root', 'root:sub', 'admin']),
                    async (_: any, {data}: {data: DeepPartial<CarEntity>}, { req }: { req: Request }): Promise<Boolean> => {
                        return !!(await CarService.createCar(data, req));
                    }
                )
            }
        }

- Thông tin thêm:
    - `schema` của GraphQL mục đích chính chỉ là xác định hình dáng của dữ liệu trong project. `schema` không thực sự có thể
    
    tự xác định việc lấy dữ liệu từ đâu (database, API bên thứ 3,...), xử lý yêu cầu từ dùng như thế nào.
    
    Để thực hiện các thao tác với dữ liệu `GraphQL` phải thông qua resolvers
    
    resolvers sẽ nhận các thông tin được người dùng gửi đến và trả về dữ liệu cụ thể nào đó (đã được xác định tại `schema`).
    
    - `resolvers` bao gồm các function có tên giống với các field đã định định nghĩa tại `schema`
    
    - Function combineResolvers() ở ví dụ trên là một function sẽ thực hiện lần lượt các function mà nó được cung cấp.
        -  AuthorizationUser() function kiểm tra role của người dùng đã đăng nhập, nếu không có role phù hợp.
        -  AuthenticationUser() chỉ kiểm tra là người dùng đã đăng nhập hay chưa.
    
  ### Sử dụng
  
  Để truy vấn và cập nhật chúng ta chỉ cần thông qua một endpoint duy nhất APP-HOST/graphql (chạy project ở local thì endpoint là `http://localhost:4000/graphql`)
  
  Chúng ta có thể dùng https://studio.apollographql.com/sandbox/explorer để truy cập vào endpoint `http://localhost:4000/graphql`
- Tạo mới Car 
 ![image](https://user-images.githubusercontent.com/36842493/200883601-bbaf75b4-d4db-44ce-909d-4007ce348181.png)
- Lấy thông tin Car với id
 ![image](https://user-images.githubusercontent.com/36842493/200883798-b4de233c-67fa-4e1f-a30e-55643848e333.png)
- Lấy danh sách Car
![image](https://user-images.githubusercontent.com/36842493/200883945-eddc6055-7216-4ae1-9245-6725d57e1d5d.png)





  
