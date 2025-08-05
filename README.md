# TDD
This is the third homework repo from the Software Engineering Lab

### بخش اول: کشف و اصلاح خطا

**پرسش ۱: خطاها چیستند و چرا شناسایی نشدند؟**

دو خطای منطقی در کلاس `AccountBalanceCalculator` وجود دارد. اول، متد `calculateBalance` بررسی نمی‌کند که آیا برداشت باعث موجودی منفی می‌شود، که این یک الزام است. دوم، این متد تراکنش‌ها را در `transactionHistory` ذخیره نمی‌کند. تست‌های اولیه احتمالاً شامل سناریوهای مرزی مانند برداشت با موجودی منفی یا بررسی تاریخچه نبودند.

**پرسش ۲: آزمونی برای خطا بنویسید و کد را اصلاح کنید.**

* **مورد آزمون (Java/JUnit):**

    ```java
    import org.junit.jupiter.api.Test;
    import java.util.ArrayList;
    import java.util.List;
    import static org.junit.jupiter.api.Assertions.assertEquals;
    import static org.junit.jupiter.api.Assertions.assertThrows;

    public class AccountBalanceCalculatorTest {

        @Test
        void shouldThrowExceptionForNegativeBalanceWithdrawal() {
            List<Transaction> transactions = new ArrayList<>();
            transactions.add(new Transaction(TransactionType.DEPOSIT, 100));
            transactions.add(new Transaction(TransactionType.WITHDRAWAL, 150));

            assertThrows(IllegalArgumentException.class, () -> {
                AccountBalanceCalculator.calculateBalance(transactions);
            });
        }

        @Test
        void shouldSaveTransactionHistoryAfterCalculation() {
            AccountBalanceCalculator.clearTransactionHistory();
            List<Transaction> transactions = new ArrayList<>();
            transactions.add(new Transaction(TransactionType.DEPOSIT, 100));

            AccountBalanceCalculator.calculateBalance(transactions);

            assertEquals(1, AccountBalanceCalculator.getTransactionHistory().size());
        }
    }
    ```

* **اصلاح در `AccountBalanceCalculator.java`:**

    ```java
    public static int calculateBalance(List<Transaction> transactions) {
        int balance = 0;
        clearTransactionHistory();
        for (Transaction t : transactions) {
            if (t.getType() == TransactionType.WITHDRAWAL) {
                if (balance - t.getAmount() < 0) {
                    throw new IllegalArgumentException("برداشت نمی‌تواند منجر به موجودی منفی شود.");
                }
                balance -= t.getAmount();
            } else if (t.getType() == TransactionType.DEPOSIT) {
                balance += t.getAmount();
            }
            addTransaction(t);
        }
        return balance;
    }
    ```

**پرسش ۳: نوشتن آزمون پس از کدنویسی چه مشکلاتی ایجاد می‌کند؟**

نوشتن آزمون پس از کدنویسی ممکن است پوشش ناقصی ایجاد کند، فقط مسیرهای اصلی را تست کرده و موارد مرزی یا باگ‌ها را نادیده بگیرد. این رویکرد اثربخشی تست را کاهش داده و حس کاذب صحت کد ایجاد می‌کند.

### بخش دوم: به‌کارگیری TDD

**پرسش ۴: نوشتن آزمون‌ها پیش از کدنویسی چگونه توسعه را تسهیل می‌کند؟**

TDD مشخصه‌ای واضح از عملکرد مورد نیاز ارائه می‌دهد، توسعه‌دهندگان را به پیاده‌سازی دقیق هدایت کرده، از کد غیرضروری یا نادرست جلوگیری می‌کند و تطابق با الزامات را تضمین می‌کند.

**پرسش ۵: مزایا و معایب TDD چیست؟**

**مزایا:**

* **طراحی بهتر:** کد ماژولار و کم‌ وابسته تولید می‌کند که تست و نگهداری آن آسان‌تر است.
* **کاهش باگ:** تست مداوم، باگ‌ها را زود شناسایی می‌کند.
* **اطمینان در ریفکتورینگ:** مجموعه آزمون‌های جامع، حفظ عملکرد را طی ریفکتورینگ تضمین می‌کند.

**معایب:**

* **سربار اولیه:** نوشتن آزمون‌ها در ابتدا توسعه را کند می‌کند.
* **نیاز به انضباط:** توسعه‌دهندگان باید منضبط باشند و چرخه قرمز-سبز-ریفکتور را دنبال کنند.
* **خطر آزمون‌های بی‌اهمیت:** ممکن است آزمون‌ها بیش از حد ساده باشند و منطق کد را به‌خوبی تست نکنند.