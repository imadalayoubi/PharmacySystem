using PharmacySystem.Abstracts;
using PharmacySystem.Interfaces;
using PharmacySystem.Models;
using PharmacySystem.Services;
using PharmacySystem.Utils;
using System;
namespace PharmacySystem
{
    public static class Program
    {
        public static void Main()
        {
            // عنوان النافذة
            System.Console.Title = "Pharmacy Management System";

            // إنشاء أدوات النظام
            ILogger logger = new ConsoleLogger();
            IUserInteractor io = new ConsoleInteractor();

            // إنشاء المالك الرئيسي للنظام
            Owner owner = new Owner
            {
                Name = "Main Owner",
                Phone = "0999999999"
            };

            //  إضافة البيانات التجريبية قبل تشغيل النظام
            owner.AddPerson(new Employee { Name = "Ali", Phone = "0999999999", Age = 27, Salary = 600, HireDate = DateTime.Now });
            owner.AddPerson(new Employee { Name = "Sara", Phone = "0988888888", Age = 25, Salary = 550, HireDate = DateTime.Now.AddMonths(-3) });
            owner.AddPerson(new Customer { Name = "Omar", Phone = "0977777777" });
            owner.AddPerson(new Employee
            {
                Name = "Ali Hassan",
                Phone = "0999999999",
                Age = 30,
                Salary = 800,
                HireDate = new DateTime(2023, 5, 20)
            });

            owner.AddPerson(new Employee
            {
                Name = "Sara Ahmed",
                Phone = "0988888888",
                Age = 27,
                Salary = 700,
                HireDate = new DateTime(2024, 1, 12)
            });

            owner.AddPerson(new Customer
            {
                Name = "Omar Khaled",
                Phone = "0933333333",
                EmployeeID = 1
            });

            owner.AddPerson(new Customer
            {
                Name = "Lina Samer",
                Phone = "0944444444",
                EmployeeID = 2
            });

            owner.AddItem(new WarehouseItem
            {
                MedicineName = "Paracetamol",
                Company = "Medco",
                ShippingPrice = 1500,
                ExpiryDate = new DateTime(2026, 6, 30)
            });

            owner.AddItem(new WarehouseItem
            {
                MedicineName = "Amoxicillin",
                Company = "HealthPlus",
                ShippingPrice = 2000,
                ExpiryDate = new DateTime(2025, 12, 15)
            });

            owner.AddItem(new PharmacyItem
            {
                MedicineName = "Aspirin",
                Price = 2500,
                ExpiryDate = new DateTime(2025, 8, 1),
                Location = "Shelf A1",
                EmployeeID = 1
            });

            owner.AddItem(new PharmacyItem
            {
                MedicineName = "Ibuprofen",
                Price = 3200,
                ExpiryDate = new DateTime(2026, 3, 10),
                Location = "Shelf B2",
                EmployeeID = 2
            });

            owner.AddPurchase(new Purchase
            {
                MedicineName = "Amoxicillin",
                QuantityPurchased = 200,
                PurchasePrice = 400000,
                PurchaseDate = new DateTime(2025, 1, 25)
            });

            owner.AddPurchase(new Purchase
            {
                MedicineName = "Paracetamol",
                QuantityPurchased = 150,
                PurchasePrice = 225000,
                PurchaseDate = new DateTime(2025, 2, 12)
            });

            owner.AddSale(new SaleRecord
            {
                MedicineName = "Aspirin",
                QuantitySold = 10,
                UnitPrice = 2500,
                EmployeeID = 1
            });

            owner.AddSale(new SaleRecord
            {
                MedicineName = "Ibuprofen",
                QuantitySold = 5,
                UnitPrice = 3200,
                EmployeeID = 2
            });

            // إنشاء مركز الخدمات يشمل كلمة السر المحفوظة
            ServiceRegistry services = new ServiceRegistry(io, logger);

            //  الآن شغّل النظام بعد إدخال البيانات
            services.Run(owner);
        }
    }
        // صنف مجرد يمثل كيان شخص في النظام مثل موظف أو زبون أو مالك
    public abstract class Person : IEntity
    {
        // معرف فريد للشخص
        public int Id { get; protected set; }

        // اسم الشخص
        public string Name { get; set; }

        // رقم الهاتف
        public string Phone { get; set; }

        // دالة لعرض المعلومات، يجب تنفيذها في الأصناف المشتقة
        public abstract void ShowInfo();
    }

    // صنف مجرد يمثل كيان دواء أو منتج في النظام مثل أدوية الصيدلية أو المخزن
    public abstract class ItemBase : IEntity
    {
        // معرف فريد للعنصر
        public int Id { get; protected set; }

        // اسم الدواء
        public string MedicineName { get; set; }

        // تاريخ انتهاء الصلاحية
        public DateTime ExpiryDate { get; set; }

        // دالة لعرض المعلومات، يجب تنفيذها في الأصناف المشتقة
        public abstract void ShowInfo();
    }
        // صنف مجرد يمثل أساس جميع الخدمات (Services)
    public abstract class BaseService : IService
    {
        // كائن لإدارة الإدخال/الإخراج من المستخدم
        protected readonly IUserInteractor _io;

        // كائن لتسجيل الرسائل والسجلات
        protected readonly ILogger _log;

        // خاصية مجردة تمثل اسم الخدمة، يجب تنفيذها في الأصناف المشتقة
        public abstract string ServiceName { get; }

        // منشئ يحقن التبعيات الأساسية: واجهة الإدخال/الإخراج وواجهة التسجيل
        protected BaseService(IUserInteractor io, ILogger log)
        {
            _io = io ?? throw new ArgumentNullException(nameof(io));
            _log = log ?? throw new ArgumentNullException(nameof(log));
        }

        // دالة لطباعة عنوان تنسيقي في الواجهة
        protected void Title(string text)
        {
            _io.WriteLine("");
            _io.WriteLine(new string('-', 40));
            _io.WriteLine($"--- {text} ---");
            _io.WriteLine(new string('-', 40));
        }

        // دالة لطلب تأكيد من المستخدم بصيغة (نعم/لا)
        protected bool Confirm(string question)
        {
            string response = _io.ReadOption(question + " (y/n)", new[] { "y", "n" });
            return response.Equals("y", StringComparison.OrdinalIgnoreCase);
        }
    }
        // واجهة عامة لكل كيان يحتوي على معرف فريد
    public interface IIdentifiable
    {
        int Id { get; }
    }

    // واجهة عامة لأي كيان يمكن عرضه أو طباعته
    public interface IEntity : IIdentifiable
    {
        void ShowInfo();
    }
        // واجهة لتسجيل الرسائل - توفر طرق لتسجيل معلومات، تحذيرات، وأخطاء
    public interface ILogger
    {
        void Info(string message);   // تسجيل رسالة معلوماتية
        void Warn(string message);   // تسجيل تحذير
        void Error(string message);  // تسجيل خطأ
    }

    // واجهة لعمليات الإدخال والإخراج - تسهّل التعامل التفاعلي مع المستخدم
    public interface IUserInteractor
    {
        string ReadString(string prompt, bool allowEmpty = false);              // قراءة سلسلة نصية
        int ReadInt(string prompt, bool allowNegative = false);                 // قراءة عدد صحيح
        decimal ReadDecimal(string prompt, bool allowNegative = false);         // قراءة عدد عشري
        DateTime ReadDate(string prompt);                                       // قراءة تاريخ
        string ReadOption(string prompt, string[] valid);                       // قراءة خيار من قائمة
        void WriteLine(string text);                                            // طباعة سطر نصي
        string ReadLine();                                                      // قراءة سطر من المستخدم
        void Pause();
    }
        // ✅ واجهة أساسية للخدمات تحتوي فقط على اسم الخدمة
    public interface IService
    {
        string ServiceName { get; }
    }

    // ✅ واجهة لخدمة إدارة الموظفين
    public interface IEmployeeService : IService
    {
        void AddEmployee(Owner owner);              // إضافة موظف جديد
        void ListEmployees(Owner owner);            // عرض قائمة الموظفين
        void DeleteEmployee(Owner owner);           // حذف موظف
        void EditEmployee(Owner o);                 // تعديل بيانات موظف
    }

    // ✅ واجهة لخدمة إدارة العملاء
    public interface ICustomerService : IService
    {
        void AddCustomer(Owner owner, Employee emp); // إضافة عميل
        void ListCustomers(Owner owner);             // عرض العملاء
        void DeleteCustomer(Owner owner);            // حذف عميل
    }

    // ✅ واجهة لخدمة إدارة المخزون (المخزن)
    public interface IInventoryService : IService
    {
        void AddWarehouseItem(Owner owner);         // إضافة صنف للمخزن
        void ListWarehouse(Owner owner);            // عرض محتويات المخزن
        void AdjustQuantity(Owner owner);           // تعديل كمية صنف
        void DeleteWarehouseItem(Owner owner);      // حذف صنف من المخزن
    }

    // ✅ واجهة لخدمة إدارة رفوف الصيدلية
    public interface IPharmacyShelvesService : IService
    {
        void AddMedicine(Owner owner, Employee emp); // إضافة دواء
        void ListMedicines(Owner owner);             // عرض الأدوية
        void EditMedicine(Owner owner);              // تعديل بيانات دواء
        void DeleteMedicine(Owner owner);            // حذف دواء
    }

    // ✅ واجهة لخدمة إدارة المشتريات
    public interface IPurchaseService : IService
    {
        void AddPurchase(Owner owner);              // تسجيل شراء جديد
        void ListPurchases(Owner owner);            // عرض المشتريات
        void DeletePurchase(Owner owner);           // حذف عملية شراء
    }

    // ✅ واجهة لخدمة إدارة المبيعات
    public interface ISalesService : IService
    {
        void AddSale(Owner owner, Employee emp);     // تسجيل عملية بيع
        void ListSales(Owner owner);                 // عرض المبيعات
        void EditSale(Owner owner);                  // تعديل عملية بيع
        void DeleteSale(Owner owner);                // حذف عملية بيع
    }
        // ✅ ممثل (delegate) لتعريف حدث إضافة عملية بيع جديدة
    public delegate void SaleAddedHandler(object sender, SaleEventArgs e);

    // ✅ بيانات الحدث المرتبط بإضافة عملية بيع
    public class SaleEventArgs : EventArgs
    {
        // كائن البيع المرتبط بالحدث
        public SaleRecord Sale { get; }

        // التهيئة مع التحقق من القيمة
        public SaleEventArgs(SaleRecord sale)
        {
            Sale = sale ?? throw new ArgumentNullException(nameof(sale));
        }
    }

    // ✅ ممثل لتعريف حدث انخفاض المخزون
    public delegate void LowStockHandler(object sender, LowStockEventArgs e);

    // ✅ بيانات الحدث عند انخفاض كمية صنف في المخزن
    public class LowStockEventArgs : EventArgs
    {
        // الصنف الذي حدث له انخفاض
        public WarehouseItem Item { get; }

        // التهيئة مع التحقق من القيمة
        public LowStockEventArgs(WarehouseItem item)
        {
            Item = item ?? throw new ArgumentNullException(nameof(item));
        }
    }

    // ✅ ممثل لتعريف حدث إضافة عملية شراء جديدة
    public delegate void PurchaseAddedHandler(object sender, PurchaseEventArgs e);

    // ✅ بيانات الحدث المرتبط بإضافة عملية شراء
    public class PurchaseEventArgs : EventArgs
    {
        // كائن الشراء المرتبط بالحدث
        public Purchase Purchase { get; }

        // التهيئة مع التحقق من القيمة
        public PurchaseEventArgs(Purchase purchase)
        {
            Purchase = purchase ?? throw new ArgumentNullException(nameof(purchase));
        }
    }
        // ✅ صنف يمثل دواء في رفوف الصيدلية
    public class PharmacyItem : ItemBase
    {
        private decimal _price;
        private int _quantity;

        // السعر مع تحقق من القيم السالبة
        public decimal Price
        {
            get => _price;
            set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Price));
                _price = value;
            }
        }

        // الكمية مع تحقق داخلي
        public int Quantity
        {
            get => _quantity;
            private set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Quantity));
                _quantity = value;
            }
        }

        // موقع الدواء في الرفوف
        public string Location { get; set; }

        // معرف الموظف الذي أضاف الدواء
        public int EmployeeID { get; set; }

        // توليد معرف العنصر
        public PharmacyItem()
        {
            Id = IdGenerator.NextId();
        }

        // عرض معلومات الدواء
        public override void ShowInfo()
        {
            Console.WriteLine($"PharmacyItem ID:{Id}, Name:{MedicineName}, Price:{Price:C}, Expiry:{ExpiryDate:yyyy-MM-dd}, Qty:{Quantity}, Loc:{Location}, EmployeeID:{EmployeeID}");
        }

        // تعديل الكمية مع تحقق من السالب
        public void AdjustQuantity(int delta)
        {
            int newQty = Quantity + delta;
            if (newQty < 0) throw new InvalidOperationException("Quantity cannot be negative");
            Quantity = newQty;
        }
    }

    // ✅ صنف يمثل دواء في المخزن
    public class WarehouseItem : ItemBase
    {
        private decimal _shippingPrice;
        private int _quantity;

        // اسم الشركة المصنعة
        public string Company { get; set; }

        // سعر الشحن مع تحقق
        public decimal ShippingPrice
        {
            get => _shippingPrice;
            set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(ShippingPrice));
                _shippingPrice = value;
            }
        }

        // الكمية المخزنة
        public int Quantity
        {
            get => _quantity;
            private set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Quantity));
                _quantity = value;
            }
        }

        // توليد معرف العنصر
        public WarehouseItem()
        {
            Id = IdGenerator.NextId();
        }

        // عرض بيانات الدواء المخزن
        public override void ShowInfo()
        {
            Console.WriteLine($"WarehouseItem ID:{Id}, Name:{MedicineName}, Company:{Company}, Shipping:{ShippingPrice:C}, Expiry:{ExpiryDate:yyyy-MM-dd}, Qty:{Quantity}");
        }

        // تعديل الكمية مع تحقق من السالب
        public void AdjustQuantity(int delta)
        {
            int newQty = Quantity + delta;
            if (newQty < 0) throw new InvalidOperationException("Quantity cannot be negative");
            Quantity = newQty;
        }
    }
        // ✅ يمثل الموظف ويورث من Person
    public class Employee : Person
    {
        private int _age;
        private decimal _salary;

        // العمر مع تحقق من القيم السالبة
        public int Age
        {
            get => _age;
            set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Age));
                _age = value;
            }
        }

        // الراتب مع تحقق من القيم السالبة
        public decimal Salary
        {
            get => _salary;
            set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Salary));
                _salary = value;
            }
        }

        public DateTime GraduationDate { get; set; }
        public DateTime HireDate { get; set; } = DateTime.Now;

        // يولد معرف الموظف تلقائيًا
        public Employee()
        {
            Id = IdGenerator.NextId();
        }

        // عرض بيانات الموظف
        public override void ShowInfo()
        {
            Console.WriteLine($"Employee ID:{Id}, Name:{Name}, Phone:{Phone}, Age:{Age}, Salary:{Salary:C}");
        }
    }

    // ✅ يمثل الزبون ويورث من Person
    public class Customer : Person
    {
        // إجمالي المشتريات التي قام بها الزبون
        public decimal TotalPurchases { get; private set; }

        // معرف الموظف المرتبط بالزبون (اختياري)
        public int EmployeeID { get; set; }

        public Customer()
        {
            Id = IdGenerator.NextId();
        }

        // إضافة مبلغ إلى إجمالي المشتريات مع تحقق
        public void AddPurchase(decimal amount)
        {
            if (amount < 0) throw new ArgumentOutOfRangeException(nameof(amount));
            TotalPurchases += amount;
        }

        // عرض معلومات الزبون
        public override void ShowInfo()
        {
            Console.WriteLine($"Customer ID:{Id}, Name:{Name}, Phone:{Phone}, Total Purchases:{TotalPurchases:C}");
        }
    }

    // ✅ يمثل مالك النظام ويورث من Person
    public class Owner : Person
    {
        // قفل لحماية العمليات متعددة المسارات
        private readonly object _lock = new object();

        // قائمة الأشخاص (الموظفين والعملاء)
        public List<Person> People { get; private set; } = new List<Person>();

        // قائمة الأصناف في النظام (المستودع والرفوف)
        public List<ItemBase> Items { get; private set; } = new List<ItemBase>();

        // قائمة عمليات الشراء
        public List<Purchase> Purchases { get; private set; } = new List<Purchase>();

        // قائمة عمليات البيع
        public List<SaleRecord> Sales { get; private set; } = new List<SaleRecord>();

        public Owner()
        {
            Id = IdGenerator.NextId();
        }

        // عرض معلومات المالك
        public override void ShowInfo()
        {
            Console.WriteLine($"Owner ID:{Id}, Name:{Name}, Phone:{Phone}");
        }

        // إضافة شخص إلى القائمة مع تحقق وتزامن
        public void AddPerson(Person p)
        {
            if (p == null) throw new ArgumentNullException(nameof(p));
            lock (_lock) { People.Add(p); }
        }

        // إضافة صنف
        public void AddItem(ItemBase item)
        {
            if (item == null) throw new ArgumentNullException(nameof(item));
            lock (_lock) { Items.Add(item); }
        }

        // إضافة عملية شراء
        public void AddPurchase(Purchase p)
        {
            if (p == null) throw new ArgumentNullException(nameof(p));
            lock (_lock) { Purchases.Add(p); }
        }

        // إضافة عملية بيع
        public void AddSale(SaleRecord s)
        {
            if (s == null) throw new ArgumentNullException(nameof(s));
            lock (_lock) { Sales.Add(s); }
        }
    }
        // يمثل عملية شراء دواء من شركة مبيعات
    public class Purchase
    {
        // رقم الفاتورة (يتم توليده تلقائيًا)
        public int InvoiceID { get; private set; }

        // اسم الدواء
        public string MedicineName { get; set; }

        // الكمية المشتراة
        public int QuantityPurchased { get; set; }

        // سعر الشراء الإجمالي
        public decimal PurchasePrice { get; set; }

        // تاريخ الشراء (القيمة الافتراضية: اليوم الحالي)
        public DateTime PurchaseDate { get; set; } = DateTime.Now;

        // اسم الشركة الموردة
        public string Company { get; internal set; }

        // المُنشئ يحدد رقم الفاتورة تلقائيًا
        public Purchase()
        {
            InvoiceID = IdGenerator.NextId();
        }

        // عرض معلومات عملية الشراء
        public void ShowInfo()
        {
            Console.WriteLine($"Purchase Invoice:{InvoiceID}, Med:{MedicineName}, Qty:{QuantityPurchased}, Price:{PurchasePrice:C}, Date:{PurchaseDate:yyyy-MM-dd}");
        }
    }

    // يمثل سجل عملية بيع
    public class SaleRecord
    {
        // رقم العملية (يتم توليده تلقائيًا)
        public int SaleID { get; private set; }

        // خاصية ID ترجع رقم العملية
        public int Id => SaleID;

        // اسم الدواء المباع
        public string MedicineName { get; set; }

        // الكمية المباعة
        public int QuantitySold { get; set; }

        // سعر الوحدة
        public decimal UnitPrice { get; set; }

        // تاريخ البيع
        public DateTime SaleDate { get; set; } = DateTime.Now;

        // إجمالي السعر (سعر الوحدة × الكمية)
        public decimal TotalPrice => QuantitySold * UnitPrice;

        // اسم آخر لإجمالي السعر
        public decimal TotalAmount => TotalPrice;

        // رقم موظف المبيعات
        public int EmployeeID { get; set; }

        // المُنشئ يحدد رقم البيع تلقائيًا
        public SaleRecord()
        {
            SaleID = IdGenerator.NextId();
        }

        // عرض معلومات عملية البيع
        public void ShowInfo()
        {
            Console.WriteLine(
                $"SaleID:{SaleID}, Med:{MedicineName}, Qty:{QuantitySold}, Unit:{UnitPrice:C}, Total:{TotalPrice:C}, EmployeeID:{EmployeeID}, Date:{SaleDate:yyyy-MM-dd}"
            );
        }
    }
        // فئة مسؤولة عن إدارة العملاء
    public class CustomerServiceImpl : BaseService, ICustomerService
    {
        // اسم الخدمة
        public override string ServiceName => "Customer Management";

        // مُنشئ الفئة يأخذ أدوات التفاعل والتسجيل    public class ConsoleInteractor : IUserInteractor
        public CustomerServiceImpl(IUserInteractor io, ILogger log) : base(io, log) { }

        // إضافة عميل جديد
        public void AddCustomer(Owner owner, Employee emp)
        {
            Console.Clear();
            _log.Info("=== Add New Customer ===");

            // قراءة البيانات الأساسية للعميل
            string name = InputValidator.ReadValidatedName("Customer Name");
            string phone = InputValidator.ReadValidatedPhone("Phone");

            // إنشاء كائن العميل وإضافته للمالك
            var newCustomer = new Customer
            {
                Name = name,
                Phone = phone
            };

            owner.AddPerson(newCustomer);

            Console.Clear();
            _log.Info($"Customer '{newCustomer.Name}' added successfully.");

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("\nCustomer added successfully!");
            Console.ResetColor();

            Pause();
        }

        // عرض قائمة العملاء
        public void ListCustomers(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== List of Customers ===");
            Console.ResetColor();

            // استخراج جميع العملاء من الأشخاص
            var customers = owner.People.OfType<Customer>().ToList();

            if (!customers.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No customers found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // عرض بيانات كل عميل
            Console.ForegroundColor = ConsoleColor.Cyan;
            foreach (var customer in customers)
            {
                Console.WriteLine($"Type: {customer.GetType().Name}");
                Console.WriteLine($"ID: {customer.Id}");
                Console.WriteLine($"Name: {customer.Name}");
                Console.WriteLine($"Phone: {customer.Phone}");
                Console.WriteLine(new string('-', 40));
            }

            Console.ResetColor();
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"Total Customers: {customers.Count}");
            Console.ResetColor();

            Pause();
        }

        // حذف عميل من النظام
        public void DeleteCustomer(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== Delete Customer ===");
            Console.ResetColor();

            var customers = owner.People.OfType<Customer>().ToList();

            if (!customers.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No customers found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // إدخال معرف العميل للحذف
            Console.Write("Enter Customer ID to delete: ");
            if (!int.TryParse(Console.ReadLine(), out int id))
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Invalid ID format.");
                Console.ResetColor();
                Pause();
                return;
            }

            // التحقق من وجود العميل
            var customer = customers.FirstOrDefault(c => c.Id == id);
            if (customer == null)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Customer not found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // تأكيد الحذف
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.Write($"Are you sure you want to delete customer '{customer.Name}'? (Y/N): ");
            Console.ResetColor();
            var confirm = Console.ReadKey(true).Key;

            if (confirm != ConsoleKey.Y)
            {
                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.WriteLine("\nOperation canceled. Customer not deleted.");
                Console.ResetColor();
                Pause();
                return;
            }

            // حذف العميل
            owner.People.Remove(customer);

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"\nCustomer '{customer.Name}' deleted successfully.");
            Console.ResetColor();

            Pause();
        }

        // إيقاف مؤقت لواجهة المستخدم
        private static void Pause()
        {
            Console.ForegroundColor = ConsoleColor.DarkGray;
            Console.WriteLine("\nPress Enter to continue...");
            Console.ResetColor();
            Console.ReadLine();
        }
    }
        // فئة مسؤولة عن إدارة الموظفين وتنفيذ العمليات المتعلقة بهم
    public class EmployeeServiceImpl : BaseService, IEmployeeService
    {
        // اسم الخدمة المعروض في الواجهة
        public override string ServiceName => "Employee Management";

        // المُنشئ يستقبل أدوات التفاعل والتسجيل
        public EmployeeServiceImpl(IUserInteractor io, ILogger log) : base(io, log) { }

        // إضافة موظف جديد
        public void AddEmployee(Owner owner)
        {
            Console.Clear();
            _log.Info("=== Add New Employee ===");

            // قراءة البيانات المطلوبة للموظف من المستخدم
            string name = InputValidator.ReadValidatedName("Employee Name");
            string phone = InputValidator.ReadValidatedPhone("Phone");
            int age = InputValidator.ReadValidatedInt("Age");
            decimal salary = InputValidator.ReadValidatedDecimal("Salary");
            DateTime hireDate = InputValidator.ReadValidatedDate("Hire Date");

            // إنشاء كائن موظف جديد
            var newEmployee = new Employee
            {
                Name = name,
                Phone = phone,
                Age = age,
                Salary = (decimal)salary,
                HireDate = hireDate
            };

            // إضافة الموظف إلى قائمة الأشخاص
            owner.AddPerson(newEmployee);

            Console.Clear();
            _log.Info($"Employee '{newEmployee.Name}' added successfully.");

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("\nEmployee added successfully!");
            Console.ResetColor();

            Pause();
        }

        // عرض جميع الموظفين
        public void ListEmployees(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== List of Employees ===");
            Console.ResetColor();

            var employees = owner.People.OfType<Employee>().ToList();

            if (!employees.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No employees found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // طباعة بيانات كل موظف
            Console.ForegroundColor = ConsoleColor.Cyan;
            foreach (var emp in employees)
            {
                Console.WriteLine($"Type: {emp.GetType().Name}");
                Console.WriteLine($"ID: {emp.Id}");
                Console.WriteLine($"Name: {emp.Name}");
                Console.WriteLine($"Phone: {emp.Phone}");
                Console.WriteLine($"Age: {emp.Age}");
                Console.WriteLine($"Salary: {emp.Salary}");
                Console.WriteLine($"Hire Date: {emp.HireDate:yyyy-MM-dd}");
                Console.WriteLine(new string('-', 40));
            }

            Console.ResetColor();
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"Total Employees: {employees.Count}");
            Console.ResetColor();

            Pause();
        }

        // تعديل بيانات موظف محدد
        public void EditEmployee(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== Edit Employee ===");
            Console.ResetColor();

            var employees = owner.People.OfType<Employee>().ToList();
            if (!employees.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No employees found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // طلب المعرف من المستخدم وتحديد الموظف
            Console.Write("Enter Employee ID to edit: ");
            if (!int.TryParse(Console.ReadLine(), out int id))
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Invalid ID format.");
                Console.ResetColor();
                Pause();
                return;
            }

            var employee = employees.FirstOrDefault(e => e.Id == id);
            if (employee == null)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Employee not found.");
                Console.ResetColor();
                Pause();
                return;
            }

            Console.ForegroundColor = ConsoleColor.Cyan;
            Console.WriteLine($"Editing employee: {employee.Name}");
            Console.ResetColor();

            // التحقق من الإدخالات الجديدة وتعديل القيم إذا تم إدخالها
            string name = InputValidator.ReadValidatedName($"New Name (leave empty to keep '{employee.Name}')");
            if (!string.IsNullOrWhiteSpace(name)) employee.Name = name;

            string phone = InputValidator.ReadValidatedPhone($"New Phone (leave empty to keep '{employee.Phone}')");
            if (!string.IsNullOrWhiteSpace(phone)) employee.Phone = phone;

            Console.Clear();
            Console.Write($"New Age (leave empty to keep {employee.Age}): ");
            string ageInput = Console.ReadLine();
            if (int.TryParse(ageInput, out int newAge)) employee.Age = newAge;

            Console.Clear();
            Console.Write($"New Salary (leave empty to keep {employee.Salary}): ");
            string salaryInput = Console.ReadLine();
            if (decimal.TryParse(salaryInput, out decimal newSalary)) employee.Salary = newSalary;

            Console.Clear();
            Console.Write($"New Hire Date (yyyy-MM-dd) (leave empty to keep {employee.HireDate:yyyy-MM-dd}): ");
            string dateInput = Console.ReadLine();
            if (DateTime.TryParse(dateInput, out DateTime newHireDate)) employee.HireDate = newHireDate;

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("\nEmployee updated successfully!");
            Console.ResetColor();

            Pause();
        }

        // حذف موظف
        public void DeleteEmployee(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== Delete Employee ===");
            Console.ResetColor();

            var employees = owner.People.OfType<Employee>().ToList();

            if (!employees.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No employees found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // تحديد الموظف بناء على المعرف
            Console.Write("Enter Employee ID to delete: ");
            if (!int.TryParse(Console.ReadLine(), out int id))
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Invalid ID format.");
                Console.ResetColor();
                Pause();
                return;
            }

            var employee = employees.FirstOrDefault(e => e.Id == id);
            if (employee == null)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Employee not found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // تأكيد الحذف
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.Write($"Are you sure you want to delete employee '{employee.Name}'? (Y/N): ");
            Console.ResetColor();
            var confirm = Console.ReadKey(true).Key;

            if (confirm != ConsoleKey.Y)
            {
                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.WriteLine("\nOperation canceled. Employee not deleted.");
                Console.ResetColor();
                Pause();
                return;
            }

            owner.People.Remove(employee);

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"\nEmployee '{employee.Name}' deleted successfully.");
            Console.ResetColor();

            Pause();
        }

        // إيقاف مؤقت بانتظار إدخال المستخدم
        private static void Pause()
        {
            Console.ForegroundColor = ConsoleColor.DarkGray;
            Console.WriteLine("\nPress Enter to continue...");
            Console.ResetColor();
            Console.ReadLine();
        }
    }
        // فئة مسؤولة عن إدارة مخزون المستودع وتنفيذ العمليات عليه
    public class InventoryServiceImpl : BaseService, IInventoryService
    {
        // اسم الخدمة المعروضة في الواجهة
        public override string ServiceName => "Inventory Management";

        // حدث يتم إطلاقه عند انخفاض كمية المخزون لأحد العناصر
        public event LowStockHandler LowStock;

        // مُنشئ الفئة يستقبل أدوات التفاعل والتسجيل
        public InventoryServiceImpl(IUserInteractor io, ILogger log) : base(io, log) { }

        // إضافة عنصر جديد إلى مخزون المستودع
        public void AddWarehouseItem(Owner owner)
        {
            Console.Clear();
            _log.Info("=== Add Warehouse Item ===");

            // قراءة البيانات الأساسية للدواء من المستخدم
            string name = InputValidator.ReadValidatedName("Medicine Name");
            string company = InputValidator.ReadValidatedName("Company Name");
            decimal shippingPrice = InputValidator.ReadValidatedDecimal("Shipping Price");
            DateTime expiryDate = InputValidator.ReadValidatedDate("Expiry Date");
            int quantity = InputValidator.ReadValidatedInt("Quantity");

            // إنشاء كائن جديد يمثل صنف دوائي في المستودع
            var newItem = new WarehouseItem
            {
                MedicineName = name,
                Company = company,
                ShippingPrice = shippingPrice,
                ExpiryDate = expiryDate
            };

            // تعديل الكمية وإضافة الصنف إلى قائمة المالك
            newItem.AdjustQuantity(quantity);
            owner.AddItem(newItem);

            Console.Clear();
            _log.Info($"Warehouse item '{newItem.MedicineName}' added successfully.");

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("\nWarehouse item added successfully!");
            Console.ResetColor();

            Pause();
        }

        // عرض قائمة الأصناف الموجودة في المستودع
        public void ListWarehouse(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== Warehouse Inventory ===");
            Console.ResetColor();

            // استخراج الأصناف التي تنتمي إلى نوع WarehouseItem
            var items = owner.Items.OfType<WarehouseItem>().ToList();

            if (!items.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No warehouse items found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // طباعة بيانات كل صنف موجود
            Console.ForegroundColor = ConsoleColor.Cyan;
            foreach (var item in items)
            {
                Console.WriteLine($"Type: {item.GetType().Name}");
                Console.WriteLine($"ID: {item.Id}");
                Console.WriteLine($"Medicine: {item.MedicineName}");
                Console.WriteLine($"Company: {item.Company}");
                Console.WriteLine($"Shipping Price: {item.ShippingPrice}");
                Console.WriteLine($"Quantity: {item.Quantity}");
                Console.WriteLine($"Expiry: {item.ExpiryDate:yyyy-MM-dd}");
                Console.WriteLine(new string('-', 40));
            }

            Console.ResetColor();
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"Total Items: {items.Count}");
            Console.ResetColor();

            Pause();
        }

        // تعديل كمية صنف محدد في المخزون
        public void AdjustQuantity(Owner owner)
        {
            Console.Clear();
            _log.Info("=== Adjust Quantity ===");

            var items = owner.Items.OfType<WarehouseItem>().ToList();
            if (!items.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No warehouse items found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // تحديد الصنف بناءً على المعرف
            int id = InputValidator.ReadValidatedInt("Enter Item ID");
            var item = items.FirstOrDefault(x => x.Id == id);

            if (item == null)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Item not found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // قراءة التغير في الكمية وتعديله
            int delta = InputValidator.ReadValidatedInt("Quantity change (+/-)");
            item.AdjustQuantity(delta);

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"Quantity adjusted. New total: {item.Quantity}");
            Console.ResetColor();

            // إطلاق الحدث إذا انخفضت الكمية تحت الحد الأدنى
            if (item.Quantity < 5)
            {
                LowStock?.Invoke(this, new LowStockEventArgs(item));
            }

            Pause();
        }

        // حذف صنف من مخزون المستودع
        public void DeleteWarehouseItem(Owner owner)
        {
            Console.Clear();
            _log.Info("=== Delete Warehouse Item ===");

            var items = owner.Items.OfType<WarehouseItem>().ToList();

            if (!items.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No warehouse items found.");
                Console.ResetColor();
                Pause();
                return;
            }

            int id = InputValidator.ReadValidatedInt("Enter Item ID to delete");
            var item = items.FirstOrDefault(x => x.Id == id);

            if (item == null)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Item not found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // تأكيد المستخدم قبل الحذف
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.Write($"Are you sure you want to delete '{item.MedicineName}'? (Y/N): ");
            Console.ResetColor();
            var confirm = Console.ReadKey(true).Key;

            if (confirm != ConsoleKey.Y)
            {
                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.WriteLine("\nOperation canceled. Item not deleted.");
                Console.ResetColor();
                Pause();
                return;
            }

            // إزالة الصنف من قائمة المالك
            owner.Items.Remove(item);

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"\nWarehouse item '{item.MedicineName}' deleted successfully.");
            Console.ResetColor();

            Pause();
        }

        // إيقاف مؤقت في واجهة المستخدم بانتظار ضغط المستخدم على Enter
        private static void Pause()
        {
            Console.ForegroundColor = ConsoleColor.DarkGray;
            Console.WriteLine("\nPress Enter to continue...");
            Console.ResetColor();
            Console.ReadLine();
        }
    }
        // فئة مسؤولة عن إدارة الموظفين وتنفيذ العمليات المتعلقة بهم
    public class EmployeeServiceImpl : BaseService, IEmployeeService
    {
        // اسم الخدمة المعروض في الواجهة
        public override string ServiceName => "Employee Management";

        // المُنشئ يستقبل أدوات التفاعل والتسجيل
        public EmployeeServiceImpl(IUserInteractor io, ILogger log) : base(io, log) { }

        // إضافة موظف جديد
        public void AddEmployee(Owner owner)
        {
            Console.Clear();
            _log.Info("=== Add New Employee ===");

            // قراءة البيانات المطلوبة للموظف من المستخدم
            string name = InputValidator.ReadValidatedName("Employee Name");
            string phone = InputValidator.ReadValidatedPhone("Phone");
            int age = InputValidator.ReadValidatedInt("Age");
            decimal salary = InputValidator.ReadValidatedDecimal("Salary");
            DateTime hireDate = InputValidator.ReadValidatedDate("Hire Date");

            // إنشاء كائن موظف جديد
            var newEmployee = new Employee
            {
                Name = name,
                Phone = phone,
                Age = age,
                Salary = (decimal)salary,
                HireDate = hireDate
            };

            // إضافة الموظف إلى قائمة الأشخاص
            owner.AddPerson(newEmployee);

            Console.Clear();
            _log.Info($"Employee '{newEmployee.Name}' added successfully.");

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("\nEmployee added successfully!");
            Console.ResetColor();

            Pause();
        }

        // عرض جميع الموظفين
        public void ListEmployees(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== List of Employees ===");
            Console.ResetColor();

            var employees = owner.People.OfType<Employee>().ToList();

            if (!employees.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No employees found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // طباعة بيانات كل موظف
            Console.ForegroundColor = ConsoleColor.Cyan;
            foreach (var emp in employees)
            {
                Console.WriteLine($"Type: {emp.GetType().Name}");
                Console.WriteLine($"ID: {emp.Id}");
                Console.WriteLine($"Name: {emp.Name}");
                Console.WriteLine($"Phone: {emp.Phone}");
                Console.WriteLine($"Age: {emp.Age}");
                Console.WriteLine($"Salary: {emp.Salary}");
                Console.WriteLine($"Hire Date: {emp.HireDate:yyyy-MM-dd}");
                Console.WriteLine(new string('-', 40));
            }

            Console.ResetColor();
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"Total Employees: {employees.Count}");
            Console.ResetColor();

            Pause();
        }

        // تعديل بيانات موظف محدد
        public void EditEmployee(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== Edit Employee ===");
            Console.ResetColor();

            var employees = owner.People.OfType<Employee>().ToList();
            if (!employees.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No employees found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // طلب المعرف من المستخدم وتحديد الموظف
            Console.Write("Enter Employee ID to edit: ");
            if (!int.TryParse(Console.ReadLine(), out int id))
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Invalid ID format.");
                Console.ResetColor();
                Pause();
                return;
            }

            var employee = employees.FirstOrDefault(e => e.Id == id);
            if (employee == null)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Employee not found.");
                Console.ResetColor();
                Pause();
                return;
            }

            Console.ForegroundColor = ConsoleColor.Cyan;
            Console.WriteLine($"Editing employee: {employee.Name}");
            Console.ResetColor();

            // التحقق من الإدخالات الجديدة وتعديل القيم إذا تم إدخالها
            string name = InputValidator.ReadValidatedName($"New Name (leave empty to keep '{employee.Name}')");
            if (!string.IsNullOrWhiteSpace(name)) employee.Name = name;

            string phone = InputValidator.ReadValidatedPhone($"New Phone (leave empty to keep '{employee.Phone}')");
            if (!string.IsNullOrWhiteSpace(phone)) employee.Phone = phone;

            Console.Clear();
            Console.Write($"New Age (leave empty to keep {employee.Age}): ");
            string ageInput = Console.ReadLine();
            if (int.TryParse(ageInput, out int newAge)) employee.Age = newAge;

            Console.Clear();
            Console.Write($"New Salary (leave empty to keep {employee.Salary}): ");
            string salaryInput = Console.ReadLine();
            if (decimal.TryParse(salaryInput, out decimal newSalary)) employee.Salary = newSalary;

            Console.Clear();
            Console.Write($"New Hire Date (yyyy-MM-dd) (leave empty to keep {employee.HireDate:yyyy-MM-dd}): ");
            string dateInput = Console.ReadLine();
            if (DateTime.TryParse(dateInput, out DateTime newHireDate)) employee.HireDate = newHireDate;

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("\nEmployee updated successfully!");
            Console.ResetColor();

            Pause();
        }

        // حذف موظف
        public void DeleteEmployee(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== Delete Employee ===");
            Console.ResetColor();

            var employees = owner.People.OfType<Employee>().ToList();

            if (!employees.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No employees found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // تحديد الموظف بناء على المعرف
            Console.Write("Enter Employee ID to delete: ");
            if (!int.TryParse(Console.ReadLine(), out int id))
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Invalid ID format.");
                Console.ResetColor();
                Pause();
                return;
            }

            var employee = employees.FirstOrDefault(e => e.Id == id);
            if (employee == null)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Employee not found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // تأكيد الحذف
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.Write($"Are you sure you want to delete employee '{employee.Name}'? (Y/N): ");
            Console.ResetColor();
            var confirm = Console.ReadKey(true).Key;

            if (confirm != ConsoleKey.Y)
            {
                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.WriteLine("\nOperation canceled. Employee not deleted.");
                Console.ResetColor();
                Pause();
                return;
            }

            owner.People.Remove(employee);

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"\nEmployee '{employee.Name}' deleted successfully.");
            Console.ResetColor();

            Pause();
        }

        // إيقاف مؤقت بانتظار إدخال المستخدم
        private static void Pause()
        {
            Console.ForegroundColor = ConsoleColor.DarkGray;
            Console.WriteLine("\nPress Enter to continue...");
            Console.ResetColor();
            Console.ReadLine();
        }
    }
        public class CustomerServiceImpl : BaseService, ICustomerService
    {
        // اسم الخدمة
        public override string ServiceName => "Customer Management";

        // مُنشئ الفئة يأخذ أدوات التفاعل والتسجيل    public class ConsoleInteractor : IUserInteractor
        public CustomerServiceImpl(IUserInteractor io, ILogger log) : base(io, log) { }

        // إضافة عميل جديد
        public void AddCustomer(Owner owner, Employee emp)
        {
            Console.Clear();
            _log.Info("=== Add New Customer ===");

            // قراءة البيانات الأساسية للعميل
            string name = InputValidator.ReadValidatedName("Customer Name");
            string phone = InputValidator.ReadValidatedPhone("Phone");

            // إنشاء كائن العميل وإضافته للمالك
            var newCustomer = new Customer
            {
                Name = name,
                Phone = phone
            };

            owner.AddPerson(newCustomer);

            Console.Clear();
            _log.Info($"Customer '{newCustomer.Name}' added successfully.");

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("\nCustomer added successfully!");
            Console.ResetColor();

            Pause();
        }

        // عرض قائمة العملاء
        public void ListCustomers(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== List of Customers ===");
            Console.ResetColor();

            // استخراج جميع العملاء من الأشخاص
            var customers = owner.People.OfType<Customer>().ToList();

            if (!customers.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No customers found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // عرض بيانات كل عميل
            Console.ForegroundColor = ConsoleColor.Cyan;
            foreach (var customer in customers)
            {
                Console.WriteLine($"Type: {customer.GetType().Name}");
                Console.WriteLine($"ID: {customer.Id}");
                Console.WriteLine($"Name: {customer.Name}");
                Console.WriteLine($"Phone: {customer.Phone}");
                Console.WriteLine(new string('-', 40));
            }

            Console.ResetColor();
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"Total Customers: {customers.Count}");
            Console.ResetColor();

            Pause();
        }

        // حذف عميل من النظام
        public void DeleteCustomer(Owner owner)
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("=== Delete Customer ===");
            Console.ResetColor();

            var customers = owner.People.OfType<Customer>().ToList();

            if (!customers.Any())
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("No customers found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // إدخال معرف العميل للحذف
            Console.Write("Enter Customer ID to delete: ");
            if (!int.TryParse(Console.ReadLine(), out int id))
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Invalid ID format.");
                Console.ResetColor();
                Pause();
                return;
            }

            // التحقق من وجود العميل
            var customer = customers.FirstOrDefault(c => c.Id == id);
            if (customer == null)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Customer not found.");
                Console.ResetColor();
                Pause();
                return;
            }

            // تأكيد الحذف
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.Write($"Are you sure you want to delete customer '{customer.Name}'? (Y/N): ");
            Console.ResetColor();
            var confirm = Console.ReadKey(true).Key;

            if (confirm != ConsoleKey.Y)
            {
                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.WriteLine("\nOperation canceled. Customer not deleted.");
                Console.ResetColor();
                Pause();
                return;
            }

            // حذف العميل
            owner.People.Remove(customer);

            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine($"\nCustomer '{customer.Name}' deleted successfully.");
            Console.ResetColor();

            Pause();
        }

        // إيقاف مؤقت لواجهة المستخدم
        private static void Pause()
        {
            Console.ForegroundColor = ConsoleColor.DarkGray;
            Console.WriteLine("\nPress Enter to continue...");
            Console.ResetColor();
            Console.ReadLine();
        }
    }
        // يمثل عملية شراء دواء من شركة مبيعات
    public class Purchase
    {
        // رقم الفاتورة (يتم توليده تلقائيًا)
        public int InvoiceID { get; private set; }

        // اسم الدواء
        public string MedicineName { get; set; }

        // الكمية المشتراة
        public int QuantityPurchased { get; set; }

        // سعر الشراء الإجمالي
        public decimal PurchasePrice { get; set; }

        // تاريخ الشراء (القيمة الافتراضية: اليوم الحالي)
        public DateTime PurchaseDate { get; set; } = DateTime.Now;

        // اسم الشركة الموردة
        public string Company { get; internal set; }

        // المُنشئ يحدد رقم الفاتورة تلقائيًا
        public Purchase()
        {
            InvoiceID = IdGenerator.NextId();
        }

        // عرض معلومات عملية الشراء
        public void ShowInfo()
        {
            Console.WriteLine($"Purchase Invoice:{InvoiceID}, Med:{MedicineName}, Qty:{QuantityPurchased}, Price:{PurchasePrice:C}, Date:{PurchaseDate:yyyy-MM-dd}");
        }
    }

    // يمثل سجل عملية بيع
    public class SaleRecord
    {
        // رقم العملية (يتم توليده تلقائيًا)
        public int SaleID { get; private set; }

        // خاصية ID ترجع رقم العملية
        public int Id => SaleID;

        // اسم الدواء المباع
        public string MedicineName { get; set; }

        // الكمية المباعة
        public int QuantitySold { get; set; }

        // سعر الوحدة
        public decimal UnitPrice { get; set; }

        // تاريخ البيع
        public DateTime SaleDate { get; set; } = DateTime.Now;

        // إجمالي السعر (سعر الوحدة × الكمية)
        public decimal TotalPrice => QuantitySold * UnitPrice;

        // اسم آخر لإجمالي السعر
        public decimal TotalAmount => TotalPrice;

        // رقم موظف المبيعات
        public int EmployeeID { get; set; }

        // المُنشئ يحدد رقم البيع تلقائيًا
        public SaleRecord()
        {
            SaleID = IdGenerator.NextId();
        }

        // عرض معلومات عملية البيع
        public void ShowInfo()
        {
            Console.WriteLine(
                $"SaleID:{SaleID}, Med:{MedicineName}, Qty:{QuantitySold}, Unit:{UnitPrice:C}, Total:{TotalPrice:C}, EmployeeID:{EmployeeID}, Date:{SaleDate:yyyy-MM-dd}"
            );
        }
    }
        // ✅ يمثل الموظف ويورث من Person
    public class Employee : Person
    {
        private int _age;
        private decimal _salary;

        // العمر مع تحقق من القيم السالبة
        public int Age
        {
            get => _age;
            set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Age));
                _age = value;
            }
        }

        // الراتب مع تحقق من القيم السالبة
        public decimal Salary
        {
            get => _salary;
            set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Salary));
                _salary = value;
            }
        }

        public DateTime GraduationDate { get; set; }
        public DateTime HireDate { get; set; } = DateTime.Now;

        // يولد معرف الموظف تلقائيًا
        public Employee()
        {
            Id = IdGenerator.NextId();
        }

        // عرض بيانات الموظف
        public override void ShowInfo()
        {
            Console.WriteLine($"Employee ID:{Id}, Name:{Name}, Phone:{Phone}, Age:{Age}, Salary:{Salary:C}");
        }
    }

    // ✅ يمثل الزبون ويورث من Person
    public class Customer : Person
    {
        // إجمالي المشتريات التي قام بها الزبون
        public decimal TotalPurchases { get; private set; }

        // معرف الموظف المرتبط بالزبون (اختياري)
        public int EmployeeID { get; set; }

        public Customer()
        {
            Id = IdGenerator.NextId();
        }

        // إضافة مبلغ إلى إجمالي المشتريات مع تحقق
        public void AddPurchase(decimal amount)
        {
            if (amount < 0) throw new ArgumentOutOfRangeException(nameof(amount));
            TotalPurchases += amount;
        }

        // عرض معلومات الزبون
        public override void ShowInfo()
        {
            Console.WriteLine($"Customer ID:{Id}, Name:{Name}, Phone:{Phone}, Total Purchases:{TotalPurchases:C}");
        }
    }

    // ✅ يمثل مالك النظام ويورث من Person
    public class Owner : Person
    {
        // قفل لحماية العمليات متعددة المسارات
        private readonly object _lock = new object();

        // قائمة الأشخاص (الموظفين والعملاء)
        public List<Person> People { get; private set; } = new List<Person>();

        // قائمة الأصناف في النظام (المستودع والرفوف)
        public List<ItemBase> Items { get; private set; } = new List<ItemBase>();

        // قائمة عمليات الشراء
        public List<Purchase> Purchases { get; private set; } = new List<Purchase>();

        // قائمة عمليات البيع
        public List<SaleRecord> Sales { get; private set; } = new List<SaleRecord>();

        public Owner()
        {
            Id = IdGenerator.NextId();
        }

        // عرض معلومات المالك
        public override void ShowInfo()
        {
            Console.WriteLine($"Owner ID:{Id}, Name:{Name}, Phone:{Phone}");
        }

        // إضافة شخص إلى القائمة مع تحقق وتزامن
        public void AddPerson(Person p)
        {
            if (p == null) throw new ArgumentNullException(nameof(p));
            lock (_lock) { People.Add(p); }
        }

        // إضافة صنف
        public void AddItem(ItemBase item)
        {
            if (item == null) throw new ArgumentNullException(nameof(item));
            lock (_lock) { Items.Add(item); }
        }

        // إضافة عملية شراء
        public void AddPurchase(Purchase p)
        {
            if (p == null) throw new ArgumentNullException(nameof(p));
            lock (_lock) { Purchases.Add(p); }
        }

        // إضافة عملية بيع
        public void AddSale(SaleRecord s)
        {
            if (s == null) throw new ArgumentNullException(nameof(s));
            lock (_lock) { Sales.Add(s); }
        }
    }
        // ✅ صنف يمثل دواء في رفوف الصيدلية
    public class PharmacyItem : ItemBase
    {
        private decimal _price;
        private int _quantity;

        // السعر مع تحقق من القيم السالبة
        public decimal Price
        {
            get => _price;
            set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Price));
                _price = value;
            }
        }

        // الكمية مع تحقق داخلي
        public int Quantity
        {
            get => _quantity;
            private set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Quantity));
                _quantity = value;
            }
        }

        // موقع الدواء في الرفوف
        public string Location { get; set; }

        // معرف الموظف الذي أضاف الدواء
        public int EmployeeID { get; set; }

        // توليد معرف العنصر
        public PharmacyItem()
        {
            Id = IdGenerator.NextId();
        }

        // عرض معلومات الدواء
        public override void ShowInfo()
        {
            Console.WriteLine($"PharmacyItem ID:{Id}, Name:{MedicineName}, Price:{Price:C}, Expiry:{ExpiryDate:yyyy-MM-dd}, Qty:{Quantity}, Loc:{Location}, EmployeeID:{EmployeeID}");
        }

        // تعديل الكمية مع تحقق من السالب
        public void AdjustQuantity(int delta)
        {
            int newQty = Quantity + delta;
            if (newQty < 0) throw new InvalidOperationException("Quantity cannot be negative");
            Quantity = newQty;
        }
    }

    // ✅ صنف يمثل دواء في المخزن
    public class WarehouseItem : ItemBase
    {
        private decimal _shippingPrice;
        private int _quantity;

        // اسم الشركة المصنعة
        public string Company { get; set; }

        // سعر الشحن مع تحقق
        public decimal ShippingPrice
        {
            get => _shippingPrice;
            set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(ShippingPrice));
                _shippingPrice = value;
            }
        }

        // الكمية المخزنة
        public int Quantity
        {
            get => _quantity;
            private set
            {
                if (value < 0) throw new ArgumentOutOfRangeException(nameof(Quantity));
                _quantity = value;
            }
        }

        // توليد معرف العنصر
        public WarehouseItem()
        {
            Id = IdGenerator.NextId();
        }

        // عرض بيانات الدواء المخزن
        public override void ShowInfo()
        {
            Console.WriteLine($"WarehouseItem ID:{Id}, Name:{MedicineName}, Company:{Company}, Shipping:{ShippingPrice:C}, Expiry:{ExpiryDate:yyyy-MM-dd}, Qty:{Quantity}");
        }

        // تعديل الكمية مع تحقق من السالب
        public void AdjustQuantity(int delta)
        {
            int newQty = Quantity + delta;
            if (newQty < 0) throw new InvalidOperationException("Quantity cannot be negative");
            Quantity = newQty;
        }
    }
        // ✅ ممثل (delegate) لتعريف حدث إضافة عملية بيع جديدة
    public delegate void SaleAddedHandler(object sender, SaleEventArgs e);

    // ✅ بيانات الحدث المرتبط بإضافة عملية بيع
    public class SaleEventArgs : EventArgs
    {
        // كائن البيع المرتبط بالحدث
        public SaleRecord Sale { get; }

        // التهيئة مع التحقق من القيمة
        public SaleEventArgs(SaleRecord sale)
        {
            Sale = sale ?? throw new ArgumentNullException(nameof(sale));
        }
    }

    // ✅ ممثل لتعريف حدث انخفاض المخزون
    public delegate void LowStockHandler(object sender, LowStockEventArgs e);

    // ✅ بيانات الحدث عند انخفاض كمية صنف في المخزن
    public class LowStockEventArgs : EventArgs
    {
        // الصنف الذي حدث له انخفاض
        public WarehouseItem Item { get; }

        // التهيئة مع التحقق من القيمة
        public LowStockEventArgs(WarehouseItem item)
        {
            Item = item ?? throw new ArgumentNullException(nameof(item));
        }
    }

    // ✅ ممثل لتعريف حدث إضافة عملية شراء جديدة
    public delegate void PurchaseAddedHandler(object sender, PurchaseEventArgs e);

    // ✅ بيانات الحدث المرتبط بإضافة عملية شراء
    public class PurchaseEventArgs : EventArgs
    {
        // كائن الشراء المرتبط بالحدث
        public Purchase Purchase { get; }

        // التهيئة مع التحقق من القيمة
        public PurchaseEventArgs(Purchase purchase)
        {
            Purchase = purchase ?? throw new ArgumentNullException(nameof(purchase));
        }
    }
        // ✅ واجهة أساسية للخدمات تحتوي فقط على اسم الخدمة
    public interface IService
    {
        string ServiceName { get; }
    }

    // ✅ واجهة لخدمة إدارة الموظفين
    public interface IEmployeeService : IService
    {
        void AddEmployee(Owner owner);              // إضافة موظف جديد
        void ListEmployees(Owner owner);            // عرض قائمة الموظفين
        void DeleteEmployee(Owner owner);           // حذف موظف
        void EditEmployee(Owner o);                 // تعديل بيانات موظف
    }

    // ✅ واجهة لخدمة إدارة العملاء
    public interface ICustomerService : IService
    {
        void AddCustomer(Owner owner, Employee emp); // إضافة عميل
        void ListCustomers(Owner owner);             // عرض العملاء
        void DeleteCustomer(Owner owner);            // حذف عميل
    }

    // ✅ واجهة لخدمة إدارة المخزون (المخزن)
    public interface IInventoryService : IService
    {
        void AddWarehouseItem(Owner owner);         // إضافة صنف للمخزن
        void ListWarehouse(Owner owner);            // عرض محتويات المخزن
        void AdjustQuantity(Owner owner);           // تعديل كمية صنف
        void DeleteWarehouseItem(Owner owner);      // حذف صنف من المخزن
    }

    // ✅ واجهة لخدمة إدارة رفوف الصيدلية
    public interface IPharmacyShelvesService : IService
    {
        void AddMedicine(Owner owner, Employee emp); // إضافة دواء
        void ListMedicines(Owner owner);             // عرض الأدوية
        void EditMedicine(Owner owner);              // تعديل بيانات دواء
        void DeleteMedicine(Owner owner);            // حذف دواء
    }

    // ✅ واجهة لخدمة إدارة المشتريات
    public interface IPurchaseService : IService
    {
        void AddPurchase(Owner owner);              // تسجيل شراء جديد
        void ListPurchases(Owner owner);            // عرض المشتريات
        void DeletePurchase(Owner owner);           // حذف عملية شراء
    }

    // ✅ واجهة لخدمة إدارة المبيعات
    public interface ISalesService : IService
    {
        void AddSale(Owner owner, Employee emp);     // تسجيل عملية بيع
        void ListSales(Owner owner);                 // عرض المبيعات
        void EditSale(Owner owner);                  // تعديل عملية بيع
        void DeleteSale(Owner owner);                // حذف عملية بيع
    }
        // واجهة لتسجيل الرسائل - توفر طرق لتسجيل معلومات، تحذيرات، وأخطاء
    public interface ILogger
    {
        void Info(string message);   // تسجيل رسالة معلوماتية
        void Warn(string message);   // تسجيل تحذير
        void Error(string message);  // تسجيل خطأ
    }

    // واجهة لعمليات الإدخال والإخراج - تسهّل التعامل التفاعلي مع المستخدم
    public interface IUserInteractor
    {
        string ReadString(string prompt, bool allowEmpty = false);              // قراءة سلسلة نصية
        int ReadInt(string prompt, bool allowNegative = false);                 // قراءة عدد صحيح
        decimal ReadDecimal(string prompt, bool allowNegative = false);         // قراءة عدد عشري
        DateTime ReadDate(string prompt);                                       // قراءة تاريخ
        string ReadOption(string prompt, string[] valid);                       // قراءة خيار من قائمة
        void WriteLine(string text);                                            // طباعة سطر نصي
        string ReadLine();                                                      // قراءة سطر من المستخدم
        void Pause();
    }
        // واجهة عامة لكل كيان يحتوي على معرف فريد
    public interface IIdentifiable
    {
        int Id { get; }
    }

    // واجهة عامة لأي كيان يمكن عرضه أو طباعته
    public interface IEntity : IIdentifiable
    {
        void ShowInfo();
    }
        // صنف مجرد يمثل أساس جميع الخدمات (Services)
    public abstract class BaseService : IService
    {
        // كائن لإدارة الإدخال/الإخراج من المستخدم
        protected readonly IUserInteractor _io;

        // كائن لتسجيل الرسائل والسجلات
        protected readonly ILogger _log;

        // خاصية مجردة تمثل اسم الخدمة، يجب تنفيذها في الأصناف المشتقة
        public abstract string ServiceName { get; }

        // منشئ يحقن التبعيات الأساسية: واجهة الإدخال/الإخراج وواجهة التسجيل
        protected BaseService(IUserInteractor io, ILogger log)
        {
            _io = io ?? throw new ArgumentNullException(nameof(io));
            _log = log ?? throw new ArgumentNullException(nameof(log));
        }

        // دالة لطباعة عنوان تنسيقي في الواجهة
        protected void Title(string text)
        {
            _io.WriteLine("");
            _io.WriteLine(new string('-', 40));
            _io.WriteLine($"--- {text} ---");
            _io.WriteLine(new string('-', 40));
        }

        // دالة لطلب تأكيد من المستخدم بصيغة (نعم/لا)
        protected bool Confirm(string question)
        {
            string response = _io.ReadOption(question + " (y/n)", new[] { "y", "n" });
            return response.Equals("y", StringComparison.OrdinalIgnoreCase);
        }
    }
        // صنف مجرد يمثل كيان شخص في النظام مثل موظف أو زبون أو مالك
    public abstract class Person : IEntity
    {
        // معرف فريد للشخص
        public int Id { get; protected set; }

        // اسم الشخص
        public string Name { get; set; }

        // رقم الهاتف
        public string Phone { get; set; }

        // دالة لعرض المعلومات، يجب تنفيذها في الأصناف المشتقة
        public abstract void ShowInfo();
    }

    // صنف مجرد يمثل كيان دواء أو منتج في النظام مثل أدوية الصيدلية أو المخزن
    public abstract class ItemBase : IEntity
    {
        // معرف فريد للعنصر
        public int Id { get; protected set; }

        // اسم الدواء
        public string MedicineName { get; set; }

        // تاريخ انتهاء الصلاحية
        public DateTime ExpiryDate { get; set; }

        // دالة لعرض المعلومات، يجب تنفيذها في الأصناف المشتقة
        public abstract void ShowInfo();
    }
}
