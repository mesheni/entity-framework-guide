# Советы и рекомендации по Entity Framework

## Введение

Цель этого руководства - предоставить рекомендации по созданию приложений с использованием Entity Framework, демонстрируя советы и рекомендации по его использованию.

Все примеры были созданы с использованием базы данных [AdventureWorks](https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver15&tabs=ssms).

## Содержание

1. [Структура](#структура)
1. [Журнал](#журнал)
1. [Маппинг и конфигурация](#маппинг-и-конфигурация)
1. [Миграции](#миграции)
1. [Запросы](#запросы)
1. [Запись](#запись)
1. [Тесты](#тесты)

## Структура

- Мы попытались создать это руководство, используя DDD (проектирование домена). Если вы уже знакомы с DDD, то структура папок будет вам знакома.

```c#
├── Entities
│ ├── Department.cs
│ └── Employee.cs
│ └── ...
├── Infrastructure
│ ├── DataAccess
│ │ ├── Configurations
│ │ │ ├── DepartmentConfiguration.cs
│ │ │ ├── EmployeeConfiguration.cs
│ │ │ ├── ...
│ │ ├── Conventions
│ │ │ ├── EntityConvention.cs
│ │ │ ├── MakeAllStringsNonUnicodeConvention.cs
│ │ │ ├── ...
│ │ ├── Migrations
│ │ ├── DataContext.cs
```

> Узнайте больше о [проектировании домена](https://en.wikipedia.org/wiki/Domain-driven_design)

- Не используйте шаблоны репозитория или единицы работы, чтобы попытаться решить проблемы производительности с помощью этих шаблонов, это сложно. Чтобы не повторять других людей, которые также рекомендуют не использовать эти шаблоны, я покажу вам несколько хороших статей по этой теме:

- [Репозиторий - это новый синглтон](https://ayende.com/blog/3955/repository-is-the-new-singleton)
- [Ночь живых репозиториев](https://ayende.com/blog/3973/night-of-the-living-repositories)
- [Ложный миф об инкапсуляции доступа к данным в DAL](https://ayende.com/blog/4567/the-false-myth-of-encapsulating-data-access-in-the-dal)

**[Назад к началу](#содержание)**

## Журнал

- Используйте `Database.Log`, чтобы визуализировать инструкции SQL в контексте:

```c#
// Визуализация инструкций SQL в консольном приложении
context.Database.Log = Console.WriteLine

// Визуализация инструкций SQL в окне вывода Visual Studio
Database.Log = (l) => Debug.WriteLine(l);
```

> Узнайте больше о [Debug.WriteLine в окне вывода Visual Studio](<https://msdn.microsoft.com/pt-br/library/windows/desktop/ms698739(v=vs.100).aspx>)

**[Назад к началу](#содержание)**

## Маппинг и конфигурация

- Определите инициализацию базы данных как null, когда вы не используете миграции (хорошо для производственной среды).

### Пример без стратегии инициализации базы данных

```bash
Открыто соединение 26/10/2017 17:23:38 -02:00

SELECT Count(*)
FROM INFORMATION_SCHEMA.TABLES AS t
WHERE t.TABLE_SCHEMA + '.' + t.TABLE_NAME IN ('HumanResources.Department','Person.Person','HumanResources.EmployeeDepartmentHistory','HumanResources.Shift','HumanResources.JobCandidate','Person.Password','dbo.ErrorLog','HumanResources.Employee')
OR t.TABLE_NAME = 'EdmMetadata'

— Выполняется 26/10/2017 17:23:38 -02:00

— Завершено за 157 мс с результатом: 8

Закрыто соединение 26/10/2017 17:23:38 -02:00

'entity-framework-guide.vshost.exe' (CLR v4.0.30319: entity-framework-guide.vshost.exe): Загружен 'C:\Windows\Microsoft.Net\assembly\GAC_MSIL\System.Runtime.Serialization\v4.0_4.0.0.0__b77a5c561934e089\System.Runtime.Serialization.dll'. Пропущена загрузка символов. Модуль оптимизирован, и в отладчике включена опция 'Только мой код'.
Открыто соединение 26/10/2017 17:23:38 -02:00

SELECT
[GroupBy1].[A1] AS [C1]
FROM ( SELECT
COUNT(1) AS [A1]
FROM [dbo].[__MigrationHistory] AS [Extent1]
WHERE [Extent1].[ContextKey] = @p__linq__0
) AS [GroupBy1]

— p__linq__0: 'entity_framework_guide.Core.Infrastructure.DataAccess.DataContext' (Тип = Строка, Размер = 4000)

— Выполняется 26/10/2017 17:23:38 -02:00

— Неудача за 38 мс с ошибкой: Неверное имя объекта 'dbo.__MigrationHistory'.

— Закрыто соединение 26/10/2017 17:23:38 -02:00

Открыто соединение 26/10/2017 17:23:38 -02:00

SELECT
[GroupBy1].[A1] AS [C1]
FROM ( SELECT
COUNT(1) AS [A1]
FROM [dbo].[__MigrationHistory] AS [Extent1]
) AS [GroupBy1]

— Выполняется 26/10/2017 17:23:38 -02:00

— Неудача за 1 мс с ошибкой: Неверное имя объекта 'dbo.__MigrationHistory'.

— Закрыто соединение 26/10/2017 17:23:38 -02:00

'entity-framework-guide.vshost.exe' (CLR v4.0.30319: entity-framework-guide.vshost.exe): Загружен 'EntityFrameworkDynamicProxies-EntityFramework'.
Открыто соединение 26/10/2017 17:23:38 -02:00

'entity-framework-guide.vshost.exe' (CLR v4.0.30319: entity-framework-guide.vshost.exe): Загружен 'EntityFrameworkDynamicProxies-entity-framework-guide'.
SELECT
[Limit1].[C1] AS [C1],
[Limit1].[BusinessEntityID] AS [BusinessEntityID],
[Limit1].[Title] AS [Title],
[Limit1].[FirstName] AS [FirstName],
[Limit1].[MiddleName] AS [MiddleName],
[Limit1].[LastName] AS [LastName],
[Limit1].[ModifiedDate] AS [ModifiedDate],
[Limit1].[NationalIDNumber] AS [NationalIDNumber]
FROM ( SELECT TOP (1)
[Extent1].[BusinessEntityID] AS [BusinessEntityID],
[Extent1].[NationalIDNumber] AS [NationalIDNumber],
[Extent2].[Title] AS [Title],
[Extent2].[FirstName] AS [FirstName],
[Extent2].[MiddleName] AS [MiddleName],
[Extent2].[LastName] AS [LastName],
[Extent2].[ModifiedDate] AS [ModifiedDate],
'1X0X' AS [C1]
FROM [HumanResources].[Employee] AS [Extent1]
INNER JOIN [Person].[Person] AS [Extent2] ON [Extent1].[BusinessEntityID] = [Extent2].[BusinessEntityID]
) AS [Limit1]

— Выполняется 26/10/2017 17:23:39 -02:00

— Завершено за 1560 мс с результатом: SqlDataReader

— Закрыто соединение 26/10/2017 17:23:40 -02:00
```

### Example with null database initialization

```bash

SELECT
    [Limit1].[C1] AS [C1],
    [Limit1].[BusinessEntityID] AS [BusinessEntityID],
    [Limit1].[Title] AS [Title],
    [Limit1].[FirstName] AS [FirstName],
    [Limit1].[MiddleName] AS [MiddleName],
    [Limit1].[LastName] AS [LastName],
    [Limit1].[ModifiedDate] AS [ModifiedDate],
    [Limit1].[NationalIDNumber] AS [NationalIDNumber]
    FROM ( SELECT TOP (1)
        [Extent1].[BusinessEntityID] AS [BusinessEntityID],
        [Extent1].[NationalIDNumber] AS [NationalIDNumber],
        [Extent2].[Title] AS [Title],
        [Extent2].[FirstName] AS [FirstName],
        [Extent2].[MiddleName] AS [MiddleName],
        [Extent2].[LastName] AS [LastName],
        [Extent2].[ModifiedDate] AS [ModifiedDate],
        '1X0X' AS [C1]
        FROM  [HumanResources].[Employee] AS [Extent1]
        INNER JOIN [Person].[Person] AS [Extent2] ON [Extent1].[BusinessEntityID] = [Extent2].[BusinessEntityID]
    )  AS [Limit1]


-- Executing at 26/10/2017 17:25:23 -02:00

-- Completed in 0 ms with result: SqlDataReader


```

- Не используйте стратегию инициализации базы данных внутри контекста, потому что сложно изменить стратегию в другой среде. Например, в интегрированных тестах мы можем создать базу данных SQL Compact для выполнения тестов, но для этого необходимо создать базу данных для всех тестов, поэтому в этом случае приложение может использовать `Database.SetInitializer<DataContext>(null)` и для тестов можно использовать `Database.SetInitializer<DataContext>(new DropCreateDatabaseAlways<DataContext>())`.

```csharp
//неправильно
public DataContext()
{
//очень неправильно
Database.SetInitializer<DataContext>(null);

Configuration.LazyLoadingEnabled = false;
Configuration.ProxyCreationEnabled = false;

Database.Log = (l) =>
{
Console.WriteLine(l); //ТОЛЬКО ДЛЯ КОНСОЛЬНОГО ПРИЛОЖЕНИЯ
Debug.WriteLine(l);
};
}

...

//хорошо
protected void Application_Start()
{
Database.SetInitializer<DataContext>(null);

AreaRegistration.RegisterAllAreas();
RegisterGlobalFilters(GlobalFilters.Filters);
RegisterRoutes(RouteTable.Routes);
...
}

```

- Используйте класс `EntityTypeConfiguration` для сопоставления ваших классов вместо кода в методе OnModelCreating. Когда у нас большое количество классов домена для настройки, каждый класс в методе OnModelCreating может стать неконтролируемым.

```csharp
// Класс сущности ErrorLog.cs
public class ErrorLog
{
    public int Id { get; set; }
    public DateTime Time { get; set; }
    public string UserName { get; set; }
    public int Number { get; set; }
    public int? Severity { get; set; }
    public int? State { get; set; }
    public string Procedure { get; set; }
    public int Line { get; set; }
    public string Message { get; set; }
}

// Класс конфигурации ErrorLogConfiguration.cs
public class ErrorLogConfiguration : EntityTypeConfiguration<ErrorLog>
{
    public ErrorLogConfiguration()
    {
        this.ToTable("ErrorLog");

        this.HasKey(t => t.Id);

        this.Property(t => t.Id)
        .HasColumnName("ErrorLogID");

        this.Property(t => t.Line)
        .HasColumnName("ErrorLine");

        this.Property(t => t.Message)
        .HasColumnName("ErrorMessage")
        .HasMaxLength(4000);

        this.Property(t => t.Number)
        .HasColumnName("ErrorNumber")
        .IsRequired();

        this.Property(t => t.Procedure)
        .HasColumnName("ErrorProcedure")
        .HasMaxLength(126);

        this.Property(t => t.Procedure)
        .HasColumnName("ErrorProcedure")
        .HasMaxLength(126);

        this.Property(t => t.Severity)
        .HasColumnName("ErrorSeverity");

        this.Property(t => t.State)
        .HasColumnName("ErrorState");

        this.Property(t => t.Time)
        .HasColumnName("ErrorTime")
        .IsRequired();

        this.Property(t => t.UserName)
        .HasColumnName("ErrorName")
        .HasMaxLength(128)
        .IsRequired();
    }
}
```

> Чтобы увидеть другие [модели сопоставления](<https://msdn.microsoft.com/en-us/library/jj591617(v=vs.113).aspx>) (например, Наследование, Разделение сущностей, Разделение таблиц)

- Используйте явное сопоставление для всех свойств и отношений. Даже несмотря на то, что использование соглашений о сопоставлении является продуктивным ресурсом, явное сопоставление позволяет нам выполнять проверку данных перед отправкой их в базу данных. Таким образом, вы можете предотвратить ошибки, такие как:

- "_"Строка или двоичные данные будут проигнорированы. Заявление было прекращено."_
- "_"Преобразование типа данных datetime2 в тип данных datetime привело к выходу значения за пределы допустимого диапазона."_

- Используйте соглашения для глобальных типов.

```csharp

// Использование типа varchar вместо nvarchar для всех типов строк
public class UnicodeConvention : Convention
{
    public UnicodeConvention()
    {
        Properties<string>().Configure(t => t.IsUnicode(false));
    }
}
```
- Используйте соглашения, чтобы избежать повторяющегося кода.

```csharp
// Соглашение о сопоставлении сущностей
public class EntityConvention : Convention
{
    public EntityConvention()
    {
        Types().Where(t => t.IsAbstract == false &&
        (
            (t.BaseType.IsGenericType && t.BaseType.GetGenericTypeDefinition() == typeof(Entity<>)) ||
            t.BaseType == typeof(Entity)
        )
        )
        .Configure(t =>
        {
            t.Property("Id").IsKey().HasColumnName(t.ClrType.Name + "ID");
        });
    }
}

//Соглашение о сопоставлении аудируемых сущностей
public class AuditableConvention : Convention
{
    public AuditableConvention()
    {
        Types().Where(t => typeof(Auditable).IsAssignableFrom(t))
        .Configure(t =>
        {
            t.Property("ModifiedDate").IsRequired();
        });
    }
}

//Соглашение о сопоставлении людей (например, Сотрудников)
public class PersonConvention : Convention
{
public PersonConvention()
{
    var personType = typeof(Person);

    Types().Where(t => t == personType || t.BaseType == personType)
    .Configure(t =>
        {
            t.Property("Id").IsKey().HasColumnName("BusinessEntityID");
        });
    }
}
```
- Создавайте свойства DbSet в вашем контексте только для классов, которые вам действительно понадобятся.

- Есть некоторые классы, для которых вы никогда не будете выполнять операции (например, представления). В этих случаях следует использовать только для чтения DbQuery, чтобы сделать их доступными.

```csharp
public virtual DbQuery<IndividualCustomer> IndividualCustomers { get { return Set<IndividualCustomer>().AsNoTracking(); } }
```

- Автоматически загружайте соглашения и конфигурации в методе модельBuilder:

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    var assembly = typeof(AppDbContext).Assembly;

    modelBuilder.Configurations.AddFromAssembly(assembly);
    modelBuilder.Conventions.AddFromAssembly(assembly);
}
```

- Когда следует использовать сложный тип, его следует инициализировать в конструкторе, чтобы избежать проблем при вставке новой записи или использовании метода прикрепления.

```csharp
//Сложный тип
public class Name
{
    public string Title { get; set; }
    public string FirstName { get; set; }
    public string MiddleName { get; set; }
    public string LastName { get; set; }

public string FullName()
{
    return String.Format("{0} {1}, {2} {3}", Title, LastName, FirstName, MiddleName);
}
}

//Использование сложного типа
public abstract class Person : Entity
{
    public Person()
    {
        //Начало сложного типа
        Name = new Name();
    }

public Name Name { get; set; }
    ...
}
```

**[Назад к началу](#содержание)**

Миграция

- Поместите класс Migrations вместе с вашими классами доступа к данным:

```bash
Enable-Migrations -MigrationsDirectory "Core\Infrastructure\DataAccess\Migrations"

```

- Сгенерируйте файл скриптов при необходимости

```bash
Update-Database -Script -SourceMigration: $InitialDatabase -TargetMigration: AddPostAbstract
```

Ссылка: https://docs.microsoft.com/en-us/ef/ef6/modeling/code-first/migrations/

### Миграции

```bash
Add-Migration InitialCreate –IgnoreChanges
```

Ссылка: https://docs.microsoft.com/en-us/ef/ef6/modeling/code-first/migrations/existing-database

**[Назад к началу](#содержание)**

### Запросы

- Отключите [прокси и ленивую загрузку](https://msdn.microsoft.com/en-us/data/jj574232.aspx), с этим вам придется вручную управлять загрузкой каждого связанного свойства:

```csharp
public DataContext()
{
    Configuration.ProxyCreationEnabled = false;
    Configuration.LazyLoadingEnabled = false;
}
```

- Используйте метод `Include`, чтобы загружать сложные свойства, когда они вам нужны:

```csharp
using System.Data.Entity; // необходимо для использования лямбда-выражения с методом Include

...

var employees = context.Employees.Include(e => e.HistoryDepartments)
                                 .Include(e => e.HistoryDepartments.Select(h => h.Department))
                                 .ToArray();
```

- Метод `Find` реализует запрос с использованием ключа отображения и всегда ищет данные в локальном кэше перед базой данных.
```csharp
// b загружается из базы данных
var a = context.Employees.Where(t => t.Id < 5).ToArray().First();
var b = context.Employees.First(1);

Console.WriteLine("Имя A: {0}", a.Name.FirstName);
Console.WriteLine("Имя B: {0}", b.Name.FirstName);

```

- Для доступа к локальному кэшу используйте свойство `Local` DbSet.

```csharp
var employees = context.Employees.Where(t => t.Id < 5).ToArray();
var employee = context.Employees.Local.FirstOrDefault();
```

- Используйте метод `AsNoTracking`, чтобы читать только в ситуациях, когда не требуется отслеживание. Когда вы его используете, контекст не кэширует, другими словами, объекты не доступны для доступа через свойства DbSets `Local`.

```csharp
var employees = context.Employees.AsNoTracking().ToArray();
```

- Используйте запросы с проекциями, чтобы загружать только требуемые данные.
```csharp
context.Employees.Select(e => new { e.Id, e.Name });
```

> Когда вы используете запросы с проекциями, вам не нужно использовать метод `AsNoTracking`.

- Используйте метод `Set`, чтобы выполнять запросы к классам, которые не отображаются в контексте.

```csharp
var resumes = context.Set<JobCandidate>().Where(j => j.Id > 2)
                                         .Select(j => j.Resume)
                                         .ToArray();
```

- Используйте метод `SelectMany`, чтобы группировать свойства коллекции:

```csharp
var jobCandidates = context.Employees.SelectMany(e => e.JobCandidates) -- JobCandidates is a collection
                                      .Where(j => j.ModifiedDate < DateTime.Today).ToArray();
```

- Объединяйте запросы, чтобы избежать ненужных соединений:

```csharp
var query = context.Employees.AsQueryable();

if (!String.IsNullOrWhiteSpace(name))
    query = query.Where(e => e.Name.FirstName.Contains(name) ||
                             e.Name.MiddleName.Contains(name) ||
                             e.Name.LastName.Contains(name));

if (!String.IsNullOrWhiteSpace(departmentName))
    query = query.Where(e => e.HistoryDepartments.Any(h => h.EndDate == null && h.Department.Name.Contains(departmentName)));

var result = query.ToArray();
```

> Entity Framework выполняет запросы только после вызова методов, таких как `Single`, `SingleOrDefault`, `First`, `FirstOrDefault`, `ToList` или `ToArray`.

- Используйте запросы с использованием значения по умолчанию null, чтобы избежать проблем при отсутствии результатов, когда используются методы Max или Min.

```csharp
var minStartDate = context.Employees.SelectMany(e => e.HistoryDepartments)
                                    .Min(h => (DateTime?)h.StartDate) ?? DateTime.Today;
```

- Используйте запросы с пагинацией с одним или двумя вызовами для улучшения производительности.
```csharp
// два запроса в базе данных
var query = context.Employees.Where(p => p.Id > 0);
var total = query.Count();

var people = query.OrderBy(p => p.Name.FirstName)
                  .Skip(0) // страница
                  .Take(10) // записи на странице
                  .ToArray();

// один запрос в базе данных
var query = context.Employees.Where(p => p.Id > 0);

var page = query.OrderBy(p => p.Name.FirstName)
                .Skip(0) // страница
                .Take(10) // записи на странице
                .Select(p => new { Name = p.Name })
                .GroupBy(p => new { Total = query.Count() })
                .First();

int total = page.Key.Total;
var people = page.Select(p => p);
```

> ПРЕДУПРЕЖДЕНИЕ: Сложные запросы с пагинацией с одним запросом могут не работать.

- Используйте свои запросы с проекциями свойств


```c#
    public class ProductListViewModel
    {
        public int Id { get; set; }
        public string Name { get; set; }

        public static Expression<Func<Product, ProductListViewModel>> Projection
        {
            get
            {
                return p => new ProductListViewModel
                {
                    Id = p.Id,
                    Name = p.Name
                };

            }
        }
    }

    ...

    var result = context.Products.Select(PhotoListItemViewModel.Projection)
                        .ToArray();
```

или

```c#
    public class ProductProjetions
    {
        public static Expression<Func<Product, dynamic>> ProductListViewModel
        {
            get
            {
                return p => new
                {
                    Id = p.Id,
                    Name = p.Name
                };

            }
        }
    }

    ...

    var result = context.Products.Select(ProductProjetions.ProductListViewModel)
                                 .ToArray();
```

**[Назад к началу](#содержание)**

## Запись

- Используйте интерфейс `IValidatableObject` для реализации пользовательской валидации. Они выполняются при вызове `SaveChanges`.

```csharp
public class Department : Entity<short>, IValidatableObject
{
    public string Name { get; set; }
    public string GroupName { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        var result = new List<ValidationResult>();

        if (Name == GroupName)
            result.Add(new ValidationResult("Name и группа имени не могут быть равны"));

        return result;
    }
}
```

- Отключите `ValidateOnSaveEnabled`, когда вам нужна производительность при записи:

```csharp
context.Configuration.ValidateOnSaveEnabled = false;
```

- Отключите `AutoDetectChangesEnabled`, когда вам нужна производительность при записи:

```csharp
context.Configuration.AutoDetectChangesEnabled = false;
```

- Получите только необходимые данные для процесса записи.

```csharp
int employeeId = 1;
short departmentId = 1;
byte shiftId = 1;

var employee = new Employee { Id = employeeId };
context.Employees.Attach(employee);
context.Entry(employee).State = EntityState.Unchanged;

// получить текущий отдел для закрытия
var oldDepartment = context.Entry(employee)
                           .Collection(e => e.HistoryDepartments)
                           .Query().FirstOrDefault(h => h.EndDate == null);
oldDepartment.EndDate = DateTime.Now;

var department = new Department { Id = departmentId };
context.Departments.Attach(department);
context.Entry(department).State = EntityState.Unchanged;

var shift = new Shift { Id = shiftId };
context.Entry(shift).State = EntityState.Unchanged;

employee.HistoryDepartments.Add(new EmployeeDepartment
{
    Department = department,
    StartDate = DateTime.Now,
    Shift = shift
});

context.SaveChanges();
```

> Используя расширение `DbSet` (`https://gist.github.com/eduardosilva/58d1f672335a6788b9cbb2c2f4e747d3`), вы можете использовать метод `GetOrAttach`

```csharp
...

// от этого
var department = new Department { Id = departmentId };
context.Departments.Attach(department);
context.Entry(department).State = EntityState.Unchanged;

// к этому
var department = context.Departments.GetLocalOrAttach(d => d.Id == departmentId, () => new Department { Id = departmentId });

...

```

- Переопределите метод `SaveChanges`, чтобы добавить операции перед отправкой данных в базу данных.

```c#
public override int SaveChanges()
{
    CheckAudit();
    return base.SaveChanges();
}

private void CheckAudit()
{
    foreach (var itemChanged in ChangeTracker.Entries())
    {
        if (!(itemChanged.State == EntityState.Added || itemChanged.State == EntityState.Modified))
            continue;

        var item = itemChanged.Entity as Auditable;
        item.ModifiedDate = DateTime.Now;
    }
}

...

// Посмотрите то же действие, используя класс ChangeTracker
// для получения дополнительной информации см. исходный код
public class AudityChangeTracker : ChangeTracker<Auditable>
{
    public override void Added(Auditable entity)
    {
        entity.ModifiedDate = DateTime.Now;
    }

    public override void Deleted(Auditable entity)
    {

    }

    public override void Updated(Auditable entity)
    {
        entity.ModifiedDate = DateTime.Now;
    }
}

...

public override int SaveChanges()
{
    CheckChanges();
    return base.SaveChanges();

}

private void CheckChanges()
{
    var trackersChange = typeof(DataContext).Assembly.GetTypes()
                                                    .Where(t => t.IsAbstract == false &&
                                                                typeof(IChangeTracker).IsAssignableFrom(t))
                                                    .Select(t =>
                                                    {
                                                        dynamic trackerChangeInstance = Activator.CreateInstance(t);
                                                        return trackerChangeInstance;
                                                    })
                                                    .Cast<IChangeTracker>()
                                                    .Select(t =>
                                                    {
                                                        t.Context = this;
                                                        return t;
                                                    })
                                                    .ToArray();

    var trackedEntities = ChangeTracker.Entries()
                                        .Where(e => trackersChange.Any(tc => tc.CanApplyTo(e.Entity)))
                                        .ToArray();

    foreach (var trackEntity in trackedEntities)
    {
        var trackers = trackersChange.Where(t => t.CanApplyTo(trackEntity.Entity)).ToList();

        trackers.ForEach(t => t.ApplyTo(trackEntity));
    }
}

```

- Используйте метод `GetValidationErrors`, чтобы получить ошибки валидации перед выполнением `SaveChanges`

```c#
var newDepartment = new Department { };
context.Departments.Add(newDepartment);

// Получить все ошибки
var errors = context.GetValidationErrors();

if (!errors.Any())
    context.SaveChanges();

foreach (var error in errors)
{
    Console.WriteLine("Entity Name: " + error.Entry.Entity.GetType().Name);
    foreach (var entityError in error.ValidationErrors)
    {
        Console.WriteLine("Property: {0} | Message {1}", entityError.PropertyName, entityError.ErrorMessage);
    }
}
```

Используйте метод `GetValidationResult`, чтобы получить ошибки из конкретного класса.

```csharp
var newDepartment = new Department { Name = "A", GroupName = "A" };
context.Departments.Add(newDepartment);

var entityErrors = context.Entry(newDepartment).GetValidationResult();

if (entityErrors.IsValid)
    context.SaveChanges();

Console.WriteLine("Entity Name: " + entityErrors.Entry.Entity.GetType().Name);
foreach (var error in entityErrors.ValidationErrors)
{
    Console.WriteLine("Property: {0} | Message {1}", error.PropertyName, error.ErrorMessage);
}

Console.ReadKey();
```

**[Назад к началу](#содержание)**

## Тесты

- Вы можете писать тесты, используя:
  - Встроенный провайдер
  - Поддельный `Context` и `DbSet`
  - Фреймворки, такие как Moq

```csharp
// Пример теста с использованием фреймворка Moq

[TestMethod]
public void Get_departments()
{
    var data = new List<Department>
    {
        new Department { Name = "BBB" },
        new Department { Name = "ZZZ" },
        new Department { Name = "AAA" },
    }.AsQueryable();

    var mockSet = new Mock<DbSet<Department>>();
    mockSet.As<IQueryable<Department>>().Setup(m => m.Provider).Returns(data.Provider);
    mockSet.As<IQueryable<Department>>().Setup(m => m.Expression).Returns(data.Expression);
    mockSet.As<IQueryable<Department>>().Setup(m => m.ElementType).Returns(data.ElementType);
    mockSet.As<IQueryable<Department>>().Setup(m => m.GetEnumerator()).Returns(data.GetEnumerator());

    var mockContext = new Mock<DataContext>();
    mockContext.Setup(c => c.Departments).Returns(mockSet.Object);

    var context = mockContext.Object;

    var blogs = context.Departments.ToList();

    Assert.AreEqual(3, blogs.Count);
    Assert.AreEqual("BBB", blogs[0].Name);
    Assert.AreEqual("ZZZ", blogs[1].Name);
    Assert.AreEqual("AAA", blogs[2].Name);
}

```


>Узнайте больше о [тестах](<https://msdn.microsoft.com/en-us/library/dn314431(v=vs.113).aspx>).

>ВАЖНО: Не пишите тесты для методов Entity Framework, пишите тесты для своих методов, используйте вышеуказанные способы для достижения этой цели.

**[Назад к началу](#содержание)**