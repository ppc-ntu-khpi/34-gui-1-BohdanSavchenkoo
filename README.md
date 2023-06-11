[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-7f7980b617ed060a017424585567c406b6ee15c891e84e1186181d67ecf80aa0.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=11323329)
# UI Lab 3
![](terminal-icon.png)
![](gui-icon.png)

Це одна з робіт, які доповнюють основний цикл лабораторних робіт #1-8 (проект **Banking**, [Netbeans](https://netbeans.org/)) з ООП.  Основна мета цих додаткових вправ - познайомитись з різними видами інтерфейсів користувача та засобами їх створення. Згадувані 'базові' роботи розміщено в [окремому репозиторії](https://github.com/liketaurus/OOP-JAVA) (якщо будете робити завдання на "4" або "5" раджу переглянути [діаграму класів](https://github.com/liketaurus/OOP-JAVA/blob/master/MyBank.png), аби розуміти які методи є у класів).

В ході першої роботи вам пропонується виконати **наступне завдання** - [Робота 3: GUI з Swing](https://github.com/ppc-ntu-khpi/GUI-Lab1-Starter/blob/master/Lab%203%20-%20SWING/Lab%203.md)
  
**Додаткове завдання** (для тих хто зробив все і прагне більшого): [дивіться тут](https://github.com/ppc-ntu-khpi/GUI-Lab1-Starter/blob/master/Lab%203%20-%20SWING/Lab%203%20-%20add.md)

Всі необхідні бібліотеки містяться у теці [jars](https://github.com/ppc-ntu-khpi/GUI-Lab1-Starter/tree/master/jars). В тому числі - всі необхідні відкомпільовані класи з робіт 1-8 - файл [MyBank.jar](https://github.com/ppc-ntu-khpi/GUI-Lab1-Starter/blob/master/jars/MyBank.jar). Файл даних лежить у теці [data](https://github.com/ppc-ntu-khpi/GUI-Lab1-Starter/tree/master/data).

---
**УВАГА! Не забуваємо здавати завдання через Google Classroom та вказувати посилання на створений для вас репозиторій!**

Також пам'ятайте, що ніхто не заважає вам редагувати файл README у вашому репозиторії😉.
А ще - дуже раджу спробувати нову фічу - інтеграцію з IDE REPL.it (хоч з таким завданням вона може й не впоратись, однак, цікаво ж!).

```java
package prprprpr;

import com.mybank.domain.Bank;
import com.mybank.domain.CheckingAccount;
import com.mybank.domain.Customer;
import com.mybank.domain.SavingsAccount;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class SWINGDemo {

    private JTextPane log;
    private JButton show;
    private JComboBox<String> clients;

    public SWINGDemo() {
        initializeComponents();
        readCustomerData("../data/test.dat");
        populateClientsComboBox();
    }

    private void initializeComponents() {
        log = new JTextPane();
        log.setContentType("text/html");
        log.setPreferredSize(new Dimension(250, 250));

        show = new JButton("Show");

        clients = new JComboBox<>();

        show.addActionListener(this::showButtonClicked);
    }

    private void readCustomerData(String filename) {
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] data = line.split("\t");
                if (data.length >= 3) {
                    String firstName = data[0].trim();
                    String lastName = data[1].trim();
                    Bank.addCustomer(firstName, lastName);
                    int numAccounts = Integer.parseInt(data[2].trim());
                    for (int i = 0; i < numAccounts; i++) {
                        line = reader.readLine();
                        String[] accountData = line.split("\t");
                        if (accountData.length >= 3) {
                            String accountType = accountData[0].trim();
                            double balance = Double.parseDouble(accountData[1].trim());
                            double interestRate = Double.parseDouble(accountData[2].trim());
                            if (accountType.equals("S")) {
                                SavingsAccount savingsAccount = new SavingsAccount(balance, interestRate);
                                Bank.getCustomer(Bank.getNumberOfCustomers() - 1).addAccount(savingsAccount);
                            } else if (accountType.equals("C")) {
                                double overdraftAmount = Double.parseDouble(accountData[2].trim());
                                CheckingAccount checkingAccount = new CheckingAccount(balance, overdraftAmount);
                                Bank.getCustomer(Bank.getNumberOfCustomers() - 1).addAccount(checkingAccount);
                            }
                        }
                    }
                }
            }
        } catch (IOException e) {
            System.out.println("Error reading customers from file: " + e.getMessage());
        }
    }

    private void populateClientsComboBox() {
        for (int i = 0; i < Bank.getNumberOfCustomers(); i++) {
            clients.addItem(Bank.getCustomer(i).getFirstName() + " " + Bank.getCustomer(i).getLastName());
        }
    }

    private void showButtonClicked(ActionEvent e) {
        Customer current = Bank.getCustomer(clients.getSelectedIndex());
        StringBuilder custInfo = new StringBuilder("<html><body>")
                .append("<h2>")
                .append(current.getFirstName())
                .append(" ")
                .append(current.getLastName())
                .append("</h2>")
                .append("<hr>")
                .append("<h3>Accounts:</h3>");

        for (int i = 0; i < current.getNumberOfAccounts(); i++) {
            String accType = current.getAccount(i) instanceof CheckingAccount ? "Checking" : "Savings";
            custInfo.append("<b>Acc Type:</b> ")
                    .append(accType)
                    .append("<br>")
                    .append("<b>Balance:</b> <font color=\"red\">$")
                    .append(current.getAccount(i).getBalance())
                    .append("</font><br><br>");

        }

        custInfo.append("</body></html>");
        log.setText(custInfo.toString());
    }

    private void createAndShowGUI() {
        JFrame frame = new JFrame("MyBank clients");
        frame.setLayout(new BorderLayout());

        JPanel cpane = new JPanel();
        cpane.setLayout(new GridLayout(1, 2));
        cpane.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

        cpane.add(clients);
        cpane.add(show);

        frame.add(cpane, BorderLayout.NORTH);
        frame.add(new JScrollPane(log), BorderLayout.CENTER);

        frame.pack();
        frame.setLocationRelativeTo(null);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setResizable(false);
        frame.setVisible(true);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            SWINGDemo demo = new SWINGDemo();
            demo.createAndShowGUI();
        });
    }
}
```
![Screenshot_2.png](screans%2FScreenshot_2.png)
![Screenshot_3.png](screans%2FScreenshot_3.png)



[![Gitter](https://badges.gitter.im/PPC-SE-2020/OOP.svg)](https://gitter.im/PPC-SE-2020/OOP?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
![](https://img.shields.io/badge/Made%20with-JAVA-red.svg)
![](https://img.shields.io/badge/Made%20with-%20Netbeans-brightgreen.svg)
![](https://img.shields.io/badge/Made%20at-PPC%20NTU%20%22KhPI%22-blue.svg) 
