# Cấu trúc
- [_env]
    - [.env.example]
- [data]
    - [backup]
    - [file]
    - [temp]
- [public]
- [src]
    - [cron_job]
    - [db]
    - [event]
    - [graphql]
    - [i18n]
    - [interfaces]
    - [libs]
    - [middleware]
    - [repotitories]
    - [routes]
    - [services]
    - [test]
    - [utils]
# Hướng dẫn

# Flow cơ bản để truy xuất/ cập nhật dữ liệu.

Tạo các file cần lượt theo các thư mục dưới đây.

[entity](#entity-httpstypeormioentities) -> [repositories](#repositories) -> [services](#services) -> [graphql]

### entity https://typeorm.io/entities
Khởi tạo class entity để ánh xạ một class với một table trong database thông qua `typeORM`

-> Tạo class Car theo path src/db/entity/`car.entity.ts`

    import { Column, Entity, PrimaryGeneratedColumn, UpdateDateColumn } from "typeorm";

    
    @Entity({name: 'car', schema: 'product', synchronize: true})
    export class Car {
        @PrimaryGeneratedColumn()
        id: number

        @Column()
        name: string

        @Column()
        color: string

        @CreateDateColumn({ type: 'timestamptz', default: () => 'CURRENT_TIMESTAMP' })
        create_at: Date

        @UpdateDateColumn({ type: 'timestamptz', default: () => 'CURRENT_TIMESTAMP' })
        update_at: Date
    }

* Thông tin: 
Thêm các decorator @Entity, @Column,... nhằm đánh dấu class, property giúp cho `typeORM` có thể hiểu được class này là một entity dùng để ánh xạ vào database
- Decorator
  - @Entity đánh dấu class để `typeORM` có thể hiểu và ánh xạ với một table ở database.
    - Một số option của decorator @Entity 
      - name tùy chọn tên table khi ánh xạ vào database
      - schema chọn schema mà class sẽ ánh xạ vào
      - synchronize đồng bộ class entity với table trong database 
        
        nếu synchronize: true thì mọi thay đổi của class đều sẽ cập nhật vào table trong database.
        
  - @Column đánh dấu đây là một cột trong table.
  - Các decorator đặc biệt:
    - @PrimaryGeneratedColumn cột khóa chính tự tăng.
    - @CreateDataColumn cột tự lấy giá trị date khi tạo mới một đối tượng.
    - @UpdateDateColumn cột tự lấy giá trị date khi cập nhật một đối tượng.

### repositories
Trong project hiện tại, chúng ta sử dụng Data Mapper pattern, ở đây mọi truy vấn và cập nhật dữ liệu điều thông qua repository.

File repositories/`repo.abstract.ts` export class RepoAbstract là một lớp trừu tượng, class sẽ khởi tạo một repository cho ta để connect đến database.

Nên để có thể truy vấn hoặc cập nhật dữ liệu chỉ cần tạo một class repository kế thừa class RepoAbstract

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

### services

Các service là một lớp nằm giữa graphql (nhận yêu cầu truy vấn/ cập nhât) và repository(có quyền truy cập vào dữ liệu).

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
        async getCarById(idCar) {
            return await CarRepository.detail({id: idCar});
        },
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



  
