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

[entity](#entity) -> [repositories] -> [services] -> [graphql]

### entity
Khởi tạo class entity để ánh xạ một class với một table trong database thông qua `typeORM`

Thêm các decorator @Entity, @Column,... nhằm đánh dấu class, property giúp cho `typeORM` có thể hiểu được class này là một entity dùng để ánh xạ vào database

vd: tạo class Car theo path src/db/entity/`car.entity.ts`

    import { Column, Entity, PrimaryGeneratedColumn, UpdateDateColumn } from "typeorm";

    @Entity({name: "car", schema: "product", synchronize: true})
    export class Car {
        @PrimaryGeneratedColumn()
        id: number

        @Column("varchar", { length: 200 })
        name: string

        @Column("varchar", { length: 10 })
        color: string

        @CreateDateColumn({ type: 'timestamptz', default: () => 'CURRENT_TIMESTAMP' })
        create_at: Date
        
        @UpdateDateColumn({ type: 'timestamptz', default: () => 'CURRENT_TIMESTAMP' })
        update_at: Date
    }

- Decorator #https://typeorm.io/entities
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
  
