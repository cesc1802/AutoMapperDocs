## AutoMapper

- Chào các bạn, chúng ta tiếp tục với [AutoMapper](https://github.com/nartc/mapper) của tác giả **Chau Tran**. Hôm nay mình sẽ giới thiệu thêm 1 API nữa trong thư viện *AutoMapper*

* Giả sử mình có entity như sau:

```javascript
@Entity('ROLE')
class Role {
  @Column('number', { primary: true, name: 'ID', precision: 20, scale: 0 })
  @AutoMap()
  id: number;

  @Column('varchar2', { name: 'name', nullable: true, length: 50 })
  @AutoMap()
  description: string | null;
}
```

```javascript
@Entity('USER')
class User {
  @Column('number', { primary: true, name: 'ID', precision: 20, scale: 0 })
  @AutoMap()
  id: number;

  @Column('varchar2', { name: 'PASSWORD', nullable: true, length: 50 })
  @AutoMap()
  password: string | null;

  @Column('nvarchar2', { name: 'USER_NAME', nullable: true, length: 100 })
  @AutoMap()
  email: string | null;

  @Column('nvarchar2', { name: 'USER_NAME', nullable: true, length: 100 })
  @AutoMap()
  firstName: string | null;

  @Column('nvarchar2', { name: 'USER_NAME', nullable: true, length: 100 })
  @AutoMap()
  lastName: string | null;

  @Column('number', { primary: true, name: 'ROLE_ID', precision: 20, scale: 0 })
  @AutoMap()
  roleId: number;

  @OneToOne(type => Role)
  @JoinColumn({name: 'ID', referencedColumnName: 'roleId'})
  @AutoMap(() => Role)
  role: Role;
}
```
* Trong trường hợp bạn muốn lấy thông tin của một user các bạn sẽ thực hiện như sau: 
  * `let userInfo = userRepo.findOne(id, {relations: ['role']});`. Khi này kết quả nhận được trông sẽ như sau: 
  ```javascript
    {
      id: 1,
      password: '123456',
      email: 'admin@gmail.com',
      firstName: 'Cesc',
      lastName: 'Nguyễn',
      roleId: 1,
      role: {
        id: 1,
        description: 'staff'
      }
    }
  ```
* Tuy nhiên, trong một vài trường hợp. Khi cái `relations` không tồn tại. Lúc này chúng ta sẽ nhận được kết quả như sau:
```javascript
    {
      id: 1,
      password: '123456',
      email: 'admin@gmail.com',
      firstName: 'Cesc',
      lastName: 'Nguyễn',
      roleId: 1,
      role: undefined
    }
```
* Khi `role: undefined` nếu chúng ta thực hiện `forMember(d => d.roleName, mapFrom(s => s.role.description))` thì sẽ bị lỗi. Vì thế chúng ta sẽ phải sử dụng API `preCondition` nữa để kiểm tra `role`

```javascript
class UserVm {
  @AutoMap()
  id: number;

  @AutoMap()
  email: string;

  @AutoMap()
  fullName: string;

  @AutoMap()
  roleName: string;
}
```

* Đừng quên `Mapper.createMap(User, UserVm);`
```javascript
Mapper.createMap(User, UserVm)
  .forMember(d => d.roleName, preCondition(d => d.role !== undefined), mapFrom(s => s.role.description))
  .forMember(d => d.fullName, mapFrom(s => `${s.firstName} ${s.lastName}`));
```
* Việc còn lại cũng chẳng còn gì. Anh em backend chỉ việc vểnh râu lên nhận kết quả thôi `let userInfo = Mapper.map(userRepo.findOne(id, { relations: ['role'] } ), UserVm);`. Lúc này kết quả trông sẽ như sau nhé:
```javascript
{
  id: 1,
  email: 'admin@gmail.com',
  fullName: 'Cesc Nguyễn',
  roleName: 'staff' // hoặc sẽ là `null` trong trường hợp role là undefined các bác nhé
}
```

* Trong bài này mình xin phép giới thiệu ngắn gọn thế này thôi ạ. Bài tiếp theo mình sẽ giới thiệu một vài API hay ho và thú vị hơn. Các bạn cùng đón đọc nhé.

##### P/S: chia sẻ thêm cho ace bạn bè gần xa được biết là tác giả của thư viện này còn rất nhiệt tình hỗ trợ nữa. Trong quá trình sử dụng nếu có gì không hiểu hoặc nhưng issue gì tác giả có thể hỗ trợ mọi người bằng video call =))