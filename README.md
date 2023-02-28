# Django, uWSGI, Nginx and Python3 in a container, using Supervisord.

This Dockerfile takes an existing Django app and deploys it for production using uWSGI and Nginx. I used most of the source code from https://github.com/dockerfiles/django-uwsgi-nginx, but I modified it to use Alpine instead of Ubuntu, the former being much lighter and in my opinion much cleaner.

I added a docker-compose file to ease deployment.


### How to deploy your application
Copy the root of your Django project into the app folder, make sure to include a `requirements.txt` file. 
You need to modify the following files:

- [uwsgi.ini]
Line module=your_app.wsgi:application. `your_app` needs to be the name of your django project, i.e. the folder name where wsgi.py lives.
- [nginx-app.conf]
Lines referring to static and media content. Use their full paths.


Then:

```
$ docker-compose build
$ docker-compose up
```

If that succeds, then `curl -vvv localhost:8113` should return something. 


@@ -0,0 +1,160 @@
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Add</class>
 <widget class="QDialog" name="Add">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>368</width>
    <height>208</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Add</string>
  </property>
  <widget class="QDialogButtonBox" name="buttonBox">
   <property name="geometry">
    <rect>
     <x>0</x>
     <y>170</y>
     <width>331</width>
     <height>32</height>
    </rect>
   </property>
   <property name="orientation">
    <enum>Qt::Horizontal</enum>
   </property>
   <property name="standardButtons">
    <set>QDialogButtonBox::Cancel|QDialogButtonBox::Ok</set>
   </property>
  </widget>
  <widget class="QLabel" name="label">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>67</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>ФИО</string>
   </property>
  </widget>
  <widget class="QLabel" name="label_2">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>50</y>
     <width>67</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>пол</string>
   </property>
  </widget>
  <widget class="QLabel" name="label_3">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>90</y>
     <width>67</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>телефон</string>
   </property>
  </widget>
  <widget class="QLabel" name="label_4">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>130</y>
     <width>141</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>дата регистрации</string>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>10</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_2">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>50</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_3">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>90</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_4">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>130</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections>
  <connection>
   <sender>buttonBox</sender>
   <signal>accepted()</signal>
   <receiver>Add</receiver>
   <slot>accept()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>248</x>
     <y>254</y>
    </hint>
    <hint type="destinationlabel">
     <x>157</x>
     <y>274</y>
    </hint>
   </hints>
  </connection>
  <connection>
   <sender>buttonBox</sender>
   <signal>rejected()</signal>
   <receiver>Add</receiver>
   <slot>reject()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>316</x>
     <y>260</y>
    </hint>
    <hint type="destinationlabel">
     <x>286</x>
     <y>274</y>
    </hint>
   </hints>
  </connection>
 </connections>
</ui>
 44  
4mv/4mv_curse/delete.cpp
@@ -0,0 +1,44 @@
#include "delete.h"
#include "ui_delete.h"

Delete::Delete(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Delete)
{
    ui->setupUi(this);
}

void Delete::accept()
{
    int num = ui->textEdit->toPlainText().toInt();

    QSqlQuery query(db);
    query.exec("SELECT num FROM clients WHERE num > 0");
    query.last();
    int _max_num = query.at() + 1;
    if (num > _max_num)
    {
        QMessageBox::information(0, "Status", "Некорректный номер");
    }
    else
    {
        query.prepare("DELETE FROM clients WHERE num = (?)");
        query.addBindValue(num);
        query.exec();
        for (auto it = 1; it <= _max_num; ++it)
        {
            if (it >= num)
            {
                query.prepare("UPDATE clients SET num = (?) WHERE num = (?)");
                query.addBindValue(it - 1);
                query.addBindValue(it);
                query.exec();
            }
        }
    }
}

Delete::~Delete()
{
    delete ui;
}
 25  
4mv/4mv_curse/delete.h
@@ -0,0 +1,25 @@
#ifndef DELETE_H
#define DELETE_H

#include "stdafx.h"

namespace Ui {
class Delete;
}

class Delete : public QDialog
{
    Q_OBJECT

public:
    explicit Delete(QWidget *parent = nullptr);
    ~Delete();

public slots:
    void accept();

private:
    Ui::Delete *ui;
};

#endif // DELETE_H
 91  
4mv/4mv_curse/delete.ui
@@ -0,0 +1,91 @@
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Delete</class>
 <widget class="QDialog" name="Delete">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>268</width>
    <height>93</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Delete</string>
  </property>
  <widget class="QDialogButtonBox" name="buttonBox">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>50</y>
     <width>231</width>
     <height>32</height>
    </rect>
   </property>
   <property name="orientation">
    <enum>Qt::Horizontal</enum>
   </property>
   <property name="standardButtons">
    <set>QDialogButtonBox::Cancel|QDialogButtonBox::Ok</set>
   </property>
  </widget>
  <widget class="QLabel" name="label">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>67</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>номер</string>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit">
   <property name="geometry">
    <rect>
     <x>90</x>
     <y>10</y>
     <width>161</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections>
  <connection>
   <sender>buttonBox</sender>
   <signal>accepted()</signal>
   <receiver>Delete</receiver>
   <slot>accept()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>248</x>
     <y>254</y>
    </hint>
    <hint type="destinationlabel">
     <x>157</x>
     <y>274</y>
    </hint>
   </hints>
  </connection>
  <connection>
   <sender>buttonBox</sender>
   <signal>rejected()</signal>
   <receiver>Delete</receiver>
   <slot>reject()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>316</x>
     <y>260</y>
    </hint>
    <hint type="destinationlabel">
     <x>286</x>
     <y>274</y>
    </hint>
   </hints>
  </connection>
 </connections>
</ui>
 10  
4mv/4mv_curse/f1.html
@@ -0,0 +1,10 @@
<html>
    <head> <meta charset="UTF-8"> </head>
    <body>
        <h1>О приложении</h1>
        <p>
            В верхней части экрана представленна строка поиска, ниже таблица клиентов(обнавляется из источника каждую секунду).
            Под таблицей кнопки манипуляции данными.
        </p>
    </body>
</html>
 15  
4mv/4mv_curse/f2.html
@@ -0,0 +1,15 @@
<html>
    <head> <meta charset="UTF-8"> </head>
    <body>
        <h1>Формат данных</h1>
        <p>
            Даты вводятся в следующем формате: ГГГГ-ММ-ДД, например 0988-01-12.
        </p>
        <p>
            Длительность абонимента вводится числом дней, номер телефона числом, соответствующим номеру.
        </p>
        <p>
            Все остальные данные вводятся в произвольном символьном формате без незначащих пробелов(без пробелов в начале или конце строки).
        </p>
    </body>
</html>
 14  
4mv/4mv_curse/f3.html
@@ -0,0 +1,14 @@
<html>
    <head> <meta charset="UTF-8"> </head>
    <body>
        <h1>Поиск</h1>
        <p>
            Изначально поиск выполняется по полю ФИО, для изменения критериев поиска следует нажать кнопку правее кнопки поиска.
        </p>
        <p>
            После нажития на эту кнопку появляется меню с настройками поиска, первые шесть галочек отвечаю каждая за поиск по
            соответствующему полю. Для поиска по периоду регистрации или периоду оплаты следует заполнить поля от и до соответствующими датами.
            Последняя галочка под названием сброс отвечает за сброс настроек поиска. Одновременно может быть активна только одна галочка.
        </p>
    </body>
</html>
 12  
4mv/4mv_curse/f4.html
@@ -0,0 +1,12 @@
<html>
    <head> <meta charset="UTF-8"> </head>
    <body>
        <h1>Добавление клиента</h1>
        <p>
            Для добавления клиента нужно нажать на кнопку добавить. В открывшемся меню необходимо заполнить все поля и нажать на кнопку ок.
        </p>
        <p>
            Некорректный ввод будет сообщен сообщением об ошибке ввода.
        </p>
    </body>
</html>
 12  
4mv/4mv_curse/f5.html
@@ -0,0 +1,12 @@
<html>
    <head> <meta charset="UTF-8"> </head>
    <body>
        <h1>Редактирование клиента</h1>
        <p>
            Для редактирования клиента нужно нажать на кнопку редактировать. В открывшемся меню необходимо заполнить все поля и нажать на кнопку ок.
        </p>
        <p>
            Некорректный ввод будет сообщен сообщением об ошибке ввода.
        </p>
    </body>
</html>
 13  
4mv/4mv_curse/f6.html
@@ -0,0 +1,13 @@
<html>
    <head> <meta charset="UTF-8"> </head>
    <body>
        <h1>Удаление клиента</h1>
        <p>
            Для удаления клиента нужно нажать на кнопку удалить. В открывшемся меню необходимо ввести номер клиента из таблицы и нажать
            на кнопку ок.
        </p>
        <p>
            Некорректный ввод будет сообщен сообщением об ошибке ввода.
        </p>
    </body>
</html>
 13  
4mv/4mv_curse/f7.html
@@ -0,0 +1,13 @@
<html>
    <head> <meta charset="UTF-8"> </head>
    <body>
        <h1>Внесение информации об оплате</h1>
        <p>
            Для внесения информации об оплате нужно нажать на кнопку оплата. В открывшемся меню необходимо ввести номер клиента из таблицы,
            дату оплаты и длительность абонимента и нажать на кнопку ок.
        </p>
        <p>
            Некорректный ввод будет сообщен сообщением об ошибке ввода.
        </p>
    </body>
</html>
 21  
4mv/4mv_curse/faq.cpp
@@ -0,0 +1,21 @@
#include "faq.h"
#include "ui_faq.h"

FAQ::FAQ(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::FAQ)
{
    ui->setupUi(this);

    connect(ui->pushButton_3, SIGNAL(clicked()), ui->textBrowser, SLOT(forward()));
    connect(ui->pushButton_2, SIGNAL(clicked()), ui->textBrowser, SLOT(backward()));
    connect(ui->pushButton, SIGNAL(clicked()), ui->textBrowser, SLOT(home()));
    connect(ui->textBrowser, SIGNAL(forwardAvailable(bool)), ui->pushButton_3, SLOT(setEnabled(bool)));
    connect(ui->textBrowser, SIGNAL(backwardAvailable(bool)), ui->pushButton_2, SLOT(setEnabled(bool)));
    ui->textBrowser->setSource(QUrl::fromLocalFile("../4mv_curse/index.html"));
}

FAQ::~FAQ()
{
    delete ui;
}
 22  
4mv/4mv_curse/faq.h
@@ -0,0 +1,22 @@
#ifndef FAQ_H
#define FAQ_H

#include "stdafx.h"

namespace Ui {
class FAQ;
}

class FAQ : public QDialog
{
    Q_OBJECT

public:
    explicit FAQ(QWidget *parent = nullptr);
    ~FAQ();

private:
    Ui::FAQ *ui;
};

#endif // FAQ_H
 71  
4mv/4mv_curse/faq.ui
@@ -0,0 +1,71 @@
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>FAQ</class>
 <widget class="QDialog" name="FAQ">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>512</width>
    <height>394</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>FAQ</string>
  </property>
  <widget class="QTextBrowser" name="textBrowser">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>491</width>
     <height>331</height>
    </rect>
   </property>
   <property name="styleSheet">
    <string notr="true">background-color: rgb(114, 159, 207);</string>
   </property>
  </widget>
  <widget class="QPushButton" name="pushButton">
   <property name="geometry">
    <rect>
     <x>20</x>
     <y>350</y>
     <width>89</width>
     <height>25</height>
    </rect>
   </property>
   <property name="text">
    <string>домой</string>
   </property>
  </widget>
  <widget class="QPushButton" name="pushButton_2">
   <property name="geometry">
    <rect>
     <x>250</x>
     <y>350</y>
     <width>89</width>
     <height>25</height>
    </rect>
   </property>
   <property name="text">
    <string>&lt;&lt;</string>
   </property>
  </widget>
  <widget class="QPushButton" name="pushButton_3">
   <property name="geometry">
    <rect>
     <x>380</x>
     <y>350</y>
     <width>89</width>
     <height>25</height>
    </rect>
   </property>
   <property name="text">
    <string>&gt;&gt;</string>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections/>
</ui>
 12  
4mv/4mv_curse/help.qrc
@@ -0,0 +1,12 @@
<RCC>
    <qresource prefix="/new/prefix1">
        <file>index.html</file>
        <file>f6.html</file>
        <file>f5.html</file>
        <file>f4.html</file>
        <file>f3.html</file>
        <file>f2.html</file>
        <file>f1.html</file>
        <file>f7.html</file>
    </qresource>
</RCC>
 16  
4mv/4mv_curse/index.html
@@ -0,0 +1,16 @@
<html>
    <head> <meta charset="UTF-8"> </head>
    <body style="background-color: rgb(114, 159, 207);">
        <h2>FAQ</h2>
        <p>
            <a href="qrc:/new/prefix1/f1.html"><p>Помощь по разделам:</p></a>
            <a href="qrc:/new/prefix1/f1.html"><p>О приложении</p></a>
            <a href="qrc:/new/prefix1/f2.html"><p>Формат данных</p></a>
            <a href="qrc:/new/prefix1/f3.html"><p>Поиск</p></a>
            <a href="qrc:/new/prefix1/f4.html"><p>Добавление клиента</p></a>
            <a href="qrc:/new/prefix1/f5.html"><p>Редактирование клиента</p></a>
            <a href="qrc:/new/prefix1/f6.html"><p>Удаление клиента</p></a>
            <a href="qrc:/new/prefix1/f7.html"><p>Внесение информации об оплате</p></a>
        </p>
    </body>
</html>
 37  
4mv/4mv_curse/main.cpp
@@ -0,0 +1,37 @@
#include "mainwindow.h"

#include <QApplication>
#include <QSplashScreen>

void loadModules(QSplashScreen* psplash)
{
    QTime time;
    time.start();
    for (int it = 0; it < 100; )
    {
        if (time.elapsed() > 40)
        {
            time.start();
            ++it;
        }
        psplash->showMessage("Загрузка: "
                             + QString::number(it) + "%",
                             Qt::AlignCenter,
                             Qt::white);
        qApp->processEvents();
    }
}

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

    QSplashScreen splash(QPixmap("../load.png"));
    splash.show();
    MainWindow w;
    loadModules(&splash);
    splash.finish(&w);
    w.show();

    return a.exec();
}
 189  
4mv/4mv_curse/mainwindow.cpp
@@ -0,0 +1,189 @@
#include "mainwindow.h"
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    QMenu *pmnuHelp = new QMenu("&Помощь");
    pmnuHelp->addAction(
        "&Помощь",
        this,
        SLOT(slotHelp()),
        QKeySequence(Qt::Key_F1)
    );
    menuBar()->addMenu(pmnuHelp);

    bg.addButton(ui->checkBox, 1);
    bg.addButton(ui->checkBox_2, 2);
    bg.addButton(ui->checkBox_3, 3);
    bg.addButton(ui->checkBox_4, 4);
    bg.addButton(ui->checkBox_5, 5);
    bg.addButton(ui->checkBox_6, 6);
    bg.addButton(ui->checkBox_7, 7);
    bg.addButton(ui->checkBox_8, 8);
    bg.addButton(ui->checkBox_9, 9);
    ui->checkBox->setVisible(false);
    ui->checkBox_2->setVisible(false);
    ui->checkBox_3->setVisible(false);
    ui->checkBox_4->setVisible(false);
    ui->checkBox_5->setVisible(false);
    ui->checkBox_6->setVisible(false);
    ui->checkBox_7->setVisible(false);
    ui->checkBox_8->setVisible(false);
    ui->label->setVisible(false);
    ui->label_2->setVisible(false);
    ui->textEdit_2->setVisible(false);
    ui->textEdit_3->setVisible(false);
    ui->columnView->setVisible(false);

    db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("../clients.db");
    if (db.open()) {
        model = new QSqlTableModel(parent, db);
        model->setTable("clients");
        model->setHeaderData(0, Qt::Horizontal, tr("#"));
        model->setHeaderData(1, Qt::Horizontal, tr("ФИО"));
        model->setHeaderData(2, Qt::Horizontal, tr("пол"));
        model->setHeaderData(3, Qt::Horizontal, tr("телефон"));
        model->setHeaderData(4, Qt::Horizontal, tr("регистрация"));
        model->setHeaderData(5, Qt::Horizontal, tr("дата оплаты"));
        model->setHeaderData(6, Qt::Horizontal, tr("длительность"));
        model->select();
        ui->tableView->horizontalHeader()->setSectionResizeMode(QHeaderView::ResizeToContents);
        ui->tableView->horizontalHeader()->setStretchLastSection(true);
        ui->tableView->setModel(model);
        ui->tableView->show();
    }
    QTimer *timer = new QTimer(this);
    connect(timer, &QTimer::timeout, [&](){model->select();} );
    timer->start(1000);
}

void MainWindow::slotHelp()
{
    FAQ *faq = new FAQ();
    faq->show();
}

void MainWindow::on_pushButton_clicked()
{
    QString str = ui->textEdit->toPlainText();
    QString search = "\0";
    model->setTable("clients");
    if (ui->checkBox->isChecked()) search += "name = ";
    if (ui->checkBox_2->isChecked()) search += "sex = ";
    if (ui->checkBox_3->isChecked()) search += "phone = ";
    if (ui->checkBox_4->isChecked()) search += "reg_date = ";
    if (ui->checkBox_5->isChecked()) search += "duration = ";
    if (ui->checkBox_6->isChecked()) search += "payment = ";
    search += '\'' + str + '\'';
    if (ui->checkBox_7->isChecked())
    {
//        model->setTable("payment");
        QString
            begin = ui->textEdit_2->toPlainText(),
            end = ui->textEdit_3->toPlainText();
//        model->select();
        search = "date(pay_date) >= date('" + begin + "') AND date(pay_date) <=  date('" + end + "')";
    }
    if (ui->checkBox_8->isChecked()) {
        QString
            begin = ui->textEdit_2->toPlainText(),
            end = ui->textEdit_3->toPlainText();
        search = "date(reg_date) >= date('" + begin + "') AND date(reg_date) <=  date('" + end + "')";
    }
    if (ui->checkBox_9->isChecked())
    {
        ui->textEdit->setText("");
        search = "";
    }

    model->setHeaderData(0, Qt::Horizontal, tr("#"));
    model->setHeaderData(1, Qt::Horizontal, tr("ФИО"));
    model->setHeaderData(2, Qt::Horizontal, tr("пол"));
    model->setHeaderData(3, Qt::Horizontal, tr("телефон"));
    model->setHeaderData(4, Qt::Horizontal, tr("регистрация"));
    model->setHeaderData(5, Qt::Horizontal, tr("дата оплаты"));
    model->setHeaderData(6, Qt::Horizontal, tr("длительность"));
    model->setFilter(search);
    model->select();
}

void MainWindow::on_pushButton_2_clicked()
{
    Add *add = new Add();
    add->show();
}

void MainWindow::on_pushButton_3_clicked()
{
    Update *update = new Update();
    update->show();
}

void MainWindow::on_pushButton_4_clicked()
{
    Delete *_delete = new Delete();
    _delete->show();
}

void MainWindow::on_pushButton_5_clicked()
{
    Payment *payment= new Payment();
    payment->show();
}

void MainWindow::on_pushButton_6_clicked()
{
    if (not hidden)
    {
        ui->checkBox->setVisible(false);
        ui->checkBox_2->setVisible(false);
        ui->checkBox_3->setVisible(false);
        ui->checkBox_4->setVisible(false);
        ui->checkBox_5->setVisible(false);
        ui->checkBox_6->setVisible(false);
        ui->checkBox_7->setVisible(false);
        ui->checkBox_8->setVisible(false);
        ui->label->setVisible(false);
        ui->label_2->setVisible(false);
        ui->textEdit_2->setVisible(false);
        ui->textEdit_3->setVisible(false);
        ui->columnView->setVisible(false);
        ui->pushButton_6->setText(">>");
        ui->tableView->move(80, 70);
        ui->tableView->resize(641, 421);
        hidden = true;
    }
    else
    {
        ui->checkBox->setVisible(true);
        ui->checkBox_2->setVisible(true);
        ui->checkBox_3->setVisible(true);
        ui->checkBox_4->setVisible(true);
        ui->checkBox_5->setVisible(true);
        ui->checkBox_6->setVisible(true);
        ui->checkBox_7->setVisible(true);
        ui->checkBox_8->setVisible(true);
        ui->label->setVisible(true);
        ui->label_2->setVisible(true);
        ui->textEdit_2->setVisible(true);
        ui->textEdit_3->setVisible(true);
        ui->columnView->setVisible(true);
        ui->pushButton_6->setText("<<");
        ui->tableView->move(80, 240);
        ui->tableView->resize(641, 251);
        hidden = false;
    }

}

MainWindow::~MainWindow()
{
    db.close();
    delete ui;
}

 34  
4mv/4mv_curse/mainwindow.h
@@ -0,0 +1,34 @@
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include "stdafx.h"
#include "add.h"
#include "delete.h"
#include "update.h"
#include "payment.h"
#include "faq.h"

QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private:
    Ui::MainWindow *ui;
private slots:
    void slotHelp();
    void on_pushButton_clicked();
    void on_pushButton_2_clicked();
    void on_pushButton_3_clicked();
    void on_pushButton_4_clicked();
    void on_pushButton_5_clicked();
    void on_pushButton_6_clicked();
};
#endif // MAINWINDOW_H
 398  
4mv/4mv_curse/mainwindow.ui
@@ -0,0 +1,398 @@
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>MainWindow</class>
 <widget class="QMainWindow" name="MainWindow">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Pool</string>
  </property>
  <property name="autoFillBackground">
   <bool>false</bool>
  </property>
  <property name="animated">
   <bool>false</bool>
  </property>
  <property name="documentMode">
   <bool>false</bool>
  </property>
  <widget class="QWidget" name="centralwidget">
   <widget class="QPushButton" name="pushButton">
    <property name="geometry">
     <rect>
      <x>570</x>
      <y>30</y>
      <width>71</width>
      <height>31</height>
     </rect>
    </property>
    <property name="text">
     <string>Поиск</string>
    </property>
   </widget>
   <widget class="QColumnView" name="columnView">
    <property name="geometry">
     <rect>
      <x>80</x>
      <y>70</y>
      <width>641</width>
      <height>161</height>
     </rect>
    </property>
    <property name="styleSheet">
     <string notr="true">background-color: rgb(114, 159, 207);</string>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox">
    <property name="geometry">
     <rect>
      <x>90</x>
      <y>90</y>
      <width>92</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>ФИО</string>
    </property>
    <property name="checked">
     <bool>true</bool>
    </property>
    <property name="autoExclusive">
     <bool>true</bool>
    </property>
    <property name="tristate">
     <bool>false</bool>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox_2">
    <property name="geometry">
     <rect>
      <x>300</x>
      <y>90</y>
      <width>92</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>Пол</string>
    </property>
    <property name="autoExclusive">
     <bool>true</bool>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox_3">
    <property name="geometry">
     <rect>
      <x>510</x>
      <y>90</y>
      <width>141</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>Номер телефона</string>
    </property>
    <property name="autoExclusive">
     <bool>true</bool>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox_4">
    <property name="geometry">
     <rect>
      <x>90</x>
      <y>120</y>
      <width>151</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>Дата регистрации</string>
    </property>
    <property name="autoExclusive">
     <bool>true</bool>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox_5">
    <property name="geometry">
     <rect>
      <x>300</x>
      <y>120</y>
      <width>121</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>Дата оплаты</string>
    </property>
    <property name="autoExclusive">
     <bool>true</bool>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox_6">
    <property name="geometry">
     <rect>
      <x>510</x>
      <y>120</y>
      <width>211</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>длительность абонимента</string>
    </property>
    <property name="autoExclusive">
     <bool>true</bool>
    </property>
   </widget>
   <widget class="QTextEdit" name="textEdit">
    <property name="geometry">
     <rect>
      <x>80</x>
      <y>30</y>
      <width>481</width>
      <height>31</height>
     </rect>
    </property>
   </widget>
   <widget class="QPushButton" name="pushButton_2">
    <property name="geometry">
     <rect>
      <x>120</x>
      <y>520</y>
      <width>111</width>
      <height>25</height>
     </rect>
    </property>
    <property name="text">
     <string>Добавить</string>
    </property>
   </widget>
   <widget class="QPushButton" name="pushButton_3">
    <property name="geometry">
     <rect>
      <x>280</x>
      <y>520</y>
      <width>111</width>
      <height>25</height>
     </rect>
    </property>
    <property name="text">
     <string>Редактировать</string>
    </property>
   </widget>
   <widget class="QPushButton" name="pushButton_4">
    <property name="geometry">
     <rect>
      <x>430</x>
      <y>520</y>
      <width>101</width>
      <height>25</height>
     </rect>
    </property>
    <property name="text">
     <string>Удалить</string>
    </property>
   </widget>
   <widget class="QPushButton" name="pushButton_5">
    <property name="geometry">
     <rect>
      <x>570</x>
      <y>520</y>
      <width>111</width>
      <height>25</height>
     </rect>
    </property>
    <property name="text">
     <string>Оплата</string>
    </property>
   </widget>
   <widget class="QTableView" name="tableView">
    <property name="geometry">
     <rect>
      <x>80</x>
      <y>70</y>
      <width>641</width>
      <height>421</height>
     </rect>
    </property>
    <property name="styleSheet">
     <string notr="true">gridline-color: rgb(78, 154, 6);
background-color: rgb(90, 138, 244);</string>
    </property>
    <property name="sizeAdjustPolicy">
     <enum>QAbstractScrollArea::AdjustToContents</enum>
    </property>
    <property name="editTriggers">
     <set>QAbstractItemView::NoEditTriggers</set>
    </property>
    <property name="dragDropOverwriteMode">
     <bool>false</bool>
    </property>
    <property name="alternatingRowColors">
     <bool>false</bool>
    </property>
    <property name="sortingEnabled">
     <bool>true</bool>
    </property>
    <property name="cornerButtonEnabled">
     <bool>false</bool>
    </property>
    <attribute name="horizontalHeaderCascadingSectionResizes">
     <bool>false</bool>
    </attribute>
    <attribute name="horizontalHeaderMinimumSectionSize">
     <number>5</number>
    </attribute>
    <attribute name="horizontalHeaderDefaultSectionSize">
     <number>50</number>
    </attribute>
    <attribute name="verticalHeaderVisible">
     <bool>false</bool>
    </attribute>
    <attribute name="verticalHeaderHighlightSections">
     <bool>false</bool>
    </attribute>
   </widget>
   <widget class="QPushButton" name="pushButton_6">
    <property name="geometry">
     <rect>
      <x>650</x>
      <y>30</y>
      <width>71</width>
      <height>31</height>
     </rect>
    </property>
    <property name="text">
     <string>&gt;&gt;</string>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox_7">
    <property name="geometry">
     <rect>
      <x>90</x>
      <y>150</y>
      <width>131</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>период оплаты</string>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox_8">
    <property name="geometry">
     <rect>
      <x>300</x>
      <y>150</y>
      <width>171</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>период регистрации</string>
    </property>
   </widget>
   <widget class="QTextEdit" name="textEdit_2">
    <property name="geometry">
     <rect>
      <x>170</x>
      <y>180</y>
      <width>171</width>
      <height>31</height>
     </rect>
    </property>
   </widget>
   <widget class="QTextEdit" name="textEdit_3">
    <property name="geometry">
     <rect>
      <x>390</x>
      <y>180</y>
      <width>171</width>
      <height>31</height>
     </rect>
    </property>
   </widget>
   <widget class="QLabel" name="label">
    <property name="geometry">
     <rect>
      <x>130</x>
      <y>180</y>
      <width>41</width>
      <height>31</height>
     </rect>
    </property>
    <property name="text">
     <string>от</string>
    </property>
   </widget>
   <widget class="QLabel" name="label_2">
    <property name="geometry">
     <rect>
      <x>360</x>
      <y>180</y>
      <width>31</width>
      <height>31</height>
     </rect>
    </property>
    <property name="text">
     <string>до</string>
    </property>
   </widget>
   <widget class="QCheckBox" name="checkBox_9">
    <property name="geometry">
     <rect>
      <x>510</x>
      <y>150</y>
      <width>92</width>
      <height>23</height>
     </rect>
    </property>
    <property name="text">
     <string>сброс</string>
    </property>
   </widget>
   <zorder>pushButton</zorder>
   <zorder>columnView</zorder>
   <zorder>checkBox</zorder>
   <zorder>checkBox_2</zorder>
   <zorder>checkBox_3</zorder>
   <zorder>checkBox_4</zorder>
   <zorder>checkBox_5</zorder>
   <zorder>checkBox_6</zorder>
   <zorder>textEdit</zorder>
   <zorder>pushButton_2</zorder>
   <zorder>pushButton_3</zorder>
   <zorder>pushButton_4</zorder>
   <zorder>pushButton_5</zorder>
   <zorder>pushButton_6</zorder>
   <zorder>checkBox_7</zorder>
   <zorder>checkBox_8</zorder>
   <zorder>textEdit_2</zorder>
   <zorder>textEdit_3</zorder>
   <zorder>label</zorder>
   <zorder>label_2</zorder>
   <zorder>checkBox_9</zorder>
   <zorder>tableView</zorder>
  </widget>
  <widget class="QMenuBar" name="menubar">
   <property name="geometry">
    <rect>
     <x>0</x>
     <y>0</y>
     <width>800</width>
     <height>22</height>
    </rect>
   </property>
  </widget>
  <widget class="QStatusBar" name="statusbar"/>
 </widget>
 <resources/>
 <connections/>
</ui>
 43  
4mv/4mv_curse/payment.cpp
@@ -0,0 +1,43 @@
#include "payment.h"
#include "ui_payment.h"

Payment::Payment(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Payment)
{
    ui->setupUi(this);
}

void Payment::accept()
{
    QString
        pay_date = ui->textEdit_2->toPlainText(),
        pay_duration = ui->textEdit_3->toPlainText();
    int num = ui->textEdit->toPlainText().toInt();
    QSqlQuery query(db);
    query.exec("SELECT num FROM clients WHERE num > 0");
    query.last();
    int _max_num = query.at() + 1;
    qDebug() << _max_num << num;
    if (num > _max_num)
    {
        QMessageBox::information(0, "Status", "Некорректный номер");
    }
    else if (pay_date == "\0" or pay_duration == "\0")
    {
        QMessageBox::information(0, "Status", "Некорректный ввод");
    }
    else
    {
        query.prepare("UPDATE clients SET pay_date = :pay_date, pay_duration = :pay_duration WHERE num = :num");
        query.bindValue(":pay_date", pay_date);
        query.bindValue(":pay_duration", pay_duration);
        query.bindValue(":num", num);
        query.exec();
    }
}

Payment::~Payment()
{
    delete ui;
}
 25  
4mv/4mv_curse/payment.h
@@ -0,0 +1,25 @@
#ifndef PAYMENT_H
#define PAYMENT_H

#include "stdafx.h"

namespace Ui {
class Payment;
}

class Payment : public QDialog
{
    Q_OBJECT

public:
    explicit Payment(QWidget *parent = nullptr);
    ~Payment();

public slots:
    void accept();

private:
    Ui::Payment *ui;
};

#endif // PAYMENT_H
 137  
4mv/4mv_curse/payment.ui
@@ -0,0 +1,137 @@
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Payment</class>
 <widget class="QDialog" name="Payment">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>465</width>
    <height>172</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Payment</string>
  </property>
  <widget class="QDialogButtonBox" name="buttonBox">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>130</y>
     <width>441</width>
     <height>32</height>
    </rect>
   </property>
   <property name="orientation">
    <enum>Qt::Horizontal</enum>
   </property>
   <property name="standardButtons">
    <set>QDialogButtonBox::Cancel|QDialogButtonBox::Ok</set>
   </property>
  </widget>
  <widget class="QLabel" name="label">
   <property name="geometry">
    <rect>
     <x>20</x>
     <y>10</y>
     <width>121</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>номер</string>
   </property>
  </widget>
  <widget class="QLabel" name="label_2">
   <property name="geometry">
    <rect>
     <x>20</x>
     <y>50</y>
     <width>121</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>дата оплаты</string>
   </property>
  </widget>
  <widget class="QLabel" name="label_3">
   <property name="geometry">
    <rect>
     <x>20</x>
     <y>90</y>
     <width>181</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>длителность абонимента</string>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit">
   <property name="geometry">
    <rect>
     <x>230</x>
     <y>10</y>
     <width>221</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_2">
   <property name="geometry">
    <rect>
     <x>230</x>
     <y>50</y>
     <width>221</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_3">
   <property name="geometry">
    <rect>
     <x>230</x>
     <y>90</y>
     <width>221</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections>
  <connection>
   <sender>buttonBox</sender>
   <signal>accepted()</signal>
   <receiver>Payment</receiver>
   <slot>accept()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>248</x>
     <y>254</y>
    </hint>
    <hint type="destinationlabel">
     <x>157</x>
     <y>274</y>
    </hint>
   </hints>
  </connection>
  <connection>
   <sender>buttonBox</sender>
   <signal>rejected()</signal>
   <receiver>Payment</receiver>
   <slot>reject()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>316</x>
     <y>260</y>
    </hint>
    <hint type="destinationlabel">
     <x>286</x>
     <y>274</y>
    </hint>
   </hints>
  </connection>
 </connections>
</ui>
 23  
4mv/4mv_curse/stdafx.h
@@ -0,0 +1,23 @@
#ifndef STDAFX_H
#define STDAFX_H

#include <bits/stdc++.h>
#include <QMainWindow>
#include <QtSql>
#include <QSqlDriver>
#include <QSqlQuery>
#include <QTextEdit>
#include <QString>
#include <QButtonGroup>
#include <QTimer>
#include <QMessageBox>
#include <QDialog>
#include <QMenu>
#include <QTextBrowser>

static QSqlDatabase db;
static QButtonGroup bg;
static QSqlTableModel *model;
static bool hidden = true;

#endif // STDAFX_H
 46  
4mv/4mv_curse/update.cpp
@@ -0,0 +1,46 @@
#include "update.h"
#include "ui_update.h"

Update::Update(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Update)
{
    ui->setupUi(this);
}

void Update::accept()
{
    QString
        name = ui->textEdit->toPlainText(),
        sex = ui->textEdit_2->toPlainText(),
        phone = ui->textEdit_3->toPlainText(),
        reg_date = ui->textEdit_4->toPlainText();
    int num = ui->textEdit_7->toPlainText().toInt();
    QSqlQuery query(db);
    query.exec("SELECT num FROM clients WHERE num > 0");
    query.last();
    int _max_num = query.at() + 1;
    if (num > _max_num)
    {
        QMessageBox::information(0, "Status", "Некорректный номер");
    }
    else if (name == "\0" or sex == "\0" or phone == "\0" or reg_date == "\0")
    {
        QMessageBox::information(0, "Status", "Некорректный ввод");
    }
    else
    {
        query.prepare("UPDATE clients SET name = :name, sex = :sex, phone = :phone, reg_date = :reg_date WHERE num = :num");
        query.bindValue(":name", name);
        query.bindValue(":sex", sex);
        query.bindValue(":phone", phone);
        query.bindValue(":reg_date", reg_date);
        query.bindValue(":num", num);
        query.exec();
    }
}

Update::~Update()
{
    delete ui;
}
 25  
4mv/4mv_curse/update.h
@@ -0,0 +1,25 @@
#ifndef UPDATE_H
#define UPDATE_H

#include "stdafx.h"

namespace Ui {
class Update;
}

class Update : public QDialog
{
    Q_OBJECT

public:
    explicit Update(QWidget *parent = nullptr);
    ~Update();

public slots:
    void accept();

private:
    Ui::Update *ui;
};

#endif // UPDATE_H
 183  
4mv/4mv_curse/update.ui
@@ -0,0 +1,183 @@
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Update</class>
 <widget class="QDialog" name="Update">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>372</width>
    <height>260</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Edit</string>
  </property>
  <widget class="QDialogButtonBox" name="buttonBox">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>220</y>
     <width>351</width>
     <height>32</height>
    </rect>
   </property>
   <property name="orientation">
    <enum>Qt::Horizontal</enum>
   </property>
   <property name="standardButtons">
    <set>QDialogButtonBox::Cancel|QDialogButtonBox::Ok</set>
   </property>
  </widget>
  <widget class="QLabel" name="label_2">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>100</y>
     <width>67</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>пол</string>
   </property>
  </widget>
  <widget class="QLabel" name="label">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>60</y>
     <width>67</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>ФИО</string>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_2">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>100</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QLabel" name="label_4">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>180</y>
     <width>131</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>дата регистрации</string>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>60</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QLabel" name="label_3">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>140</y>
     <width>67</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>телефон</string>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_4">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>180</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_3">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>140</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QTextEdit" name="textEdit_7">
   <property name="geometry">
    <rect>
     <x>160</x>
     <y>20</y>
     <width>201</width>
     <height>31</height>
    </rect>
   </property>
  </widget>
  <widget class="QLabel" name="label_7">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>20</y>
     <width>67</width>
     <height>31</height>
    </rect>
   </property>
   <property name="text">
    <string>номер</string>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections>
  <connection>
   <sender>buttonBox</sender>
   <signal>accepted()</signal>
   <receiver>Update</receiver>
   <slot>accept()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>248</x>
     <y>254</y>
    </hint>
    <hint type="destinationlabel">
     <x>157</x>
     <y>274</y>
    </hint>
   </hints>
  </connection>
  <connection>
   <sender>buttonBox</sender>
   <signal>rejected()</signal>
   <receiver>Update</receiver>
   <slot>reject()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>316</x>
     <y>260</y>
    </hint>
    <hint type="destinationlabel">
     <x>286</x>
     <y>274</y>
    </hint>
   </hints>
  </connection>
 </connections>
</ui>
 BIN +4.9 MB 
4mv/build-4mv_curse-Desktop-Debug/4mv_curse
Binary file not shown.
 568  
4mv/build-4mv_curse-Desktop-Debug/Makefile
Large diffs are not rendered by default.

 BIN +1.23 MB 
4mv/build-4mv_curse-Desktop-Debug/add.o
Binary file not shown.
 BIN +1.22 MB 
4mv/build-4mv_curse-Desktop-Debug/delete.o
Binary file not shown.
 BIN +1.2 MB 
4mv/build-4mv_curse-Desktop-Debug/faq.o
Binary file not shown.
 BIN +1.21 MB 
4mv/build-4mv_curse-Desktop-Debug/main.o
Binary file not shown.
 BIN +1.31 MB 
4mv/build-4mv_curse-Desktop-Debug/mainwindow.o
Binary file not shown.
 118  
4mv/build-4mv_curse-Desktop-Debug/moc_add.cpp
@@ -0,0 +1,118 @@
/****************************************************************************
** Meta object code from reading C++ file 'add.h'
**
** Created by: The Qt Meta Object Compiler version 67 (Qt 5.12.8)
**
** WARNING! All changes made in this file will be lost!
*****************************************************************************/

#include "../4mv_curse/add.h"
#include <QtCore/qbytearray.h>
#include <QtCore/qmetatype.h>
#if !defined(Q_MOC_OUTPUT_REVISION)
#error "The header file 'add.h' doesn't include <QObject>."
#elif Q_MOC_OUTPUT_REVISION != 67
#error "This file was generated using the moc from 5.12.8. It"
#error "cannot be used with the include files from this version of Qt."
#error "(The moc has changed too much.)"
#endif

QT_BEGIN_MOC_NAMESPACE
QT_WARNING_PUSH
QT_WARNING_DISABLE_DEPRECATED
struct qt_meta_stringdata_Add_t {
    QByteArrayData data[3];
    char stringdata0[12];
};
#define QT_MOC_LITERAL(idx, ofs, len) \
    Q_STATIC_BYTE_ARRAY_DATA_HEADER_INITIALIZER_WITH_OFFSET(len, \
    qptrdiff(offsetof(qt_meta_stringdata_Add_t, stringdata0) + ofs \
        - idx * sizeof(QByteArrayData)) \
    )
static const qt_meta_stringdata_Add_t qt_meta_stringdata_Add = {
    {
QT_MOC_LITERAL(0, 0, 3), // "Add"
QT_MOC_LITERAL(1, 4, 6), // "accept"
QT_MOC_LITERAL(2, 11, 0) // ""

    },
    "Add\0accept\0"
};
#undef QT_MOC_LITERAL

static const uint qt_meta_data_Add[] = {

 // content:
       8,       // revision
       0,       // classname
       0,    0, // classinfo
       1,   14, // methods
       0,    0, // properties
       0,    0, // enums/sets
       0,    0, // constructors
       0,       // flags
       0,       // signalCount

 // slots: name, argc, parameters, tag, flags
       1,    0,   19,    2, 0x0a /* Public */,

 // slots: parameters
    QMetaType::Void,

       0        // eod
};

void Add::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    if (_c == QMetaObject::InvokeMetaMethod) {
        auto *_t = static_cast<Add *>(_o);
        Q_UNUSED(_t)
        switch (_id) {
        case 0: _t->accept(); break;
        default: ;
        }
    }
    Q_UNUSED(_a);
}

QT_INIT_METAOBJECT const QMetaObject Add::staticMetaObject = { {
    &QDialog::staticMetaObject,
    qt_meta_stringdata_Add.data,
    qt_meta_data_Add,
    qt_static_metacall,
    nullptr,
    nullptr
} };


const QMetaObject *Add::metaObject() const
{
    return QObject::d_ptr->metaObject ? QObject::d_ptr->dynamicMetaObject() : &staticMetaObject;
}

void *Add::qt_metacast(const char *_clname)
{
    if (!_clname) return nullptr;
    if (!strcmp(_clname, qt_meta_stringdata_Add.stringdata0))
        return static_cast<void*>(this);
    return QDialog::qt_metacast(_clname);
}

int Add::qt_metacall(QMetaObject::Call _c, int _id, void **_a)
{
    _id = QDialog::qt_metacall(_c, _id, _a);
    if (_id < 0)
        return _id;
    if (_c == QMetaObject::InvokeMetaMethod) {
        if (_id < 1)
            qt_static_metacall(this, _c, _id, _a);
        _id -= 1;
    } else if (_c == QMetaObject::RegisterMethodArgumentMetaType) {
        if (_id < 1)
            *reinterpret_cast<int*>(_a[0]) = -1;
        _id -= 1;
    }
    return _id;
}
QT_WARNING_POP
QT_END_MOC_NAMESPACE
 BIN +1.19 MB 
4mv/build-4mv_curse-Desktop-Debug/moc_add.o
Binary file not shown.
 118  
4mv/build-4mv_curse-Desktop-Debug/moc_delete.cpp
@@ -0,0 +1,118 @@
/****************************************************************************
** Meta object code from reading C++ file 'delete.h'
**
** Created by: The Qt Meta Object Compiler version 67 (Qt 5.12.8)
**
** WARNING! All changes made in this file will be lost!
*****************************************************************************/

#include "../4mv_curse/delete.h"
#include <QtCore/qbytearray.h>
#include <QtCore/qmetatype.h>
#if !defined(Q_MOC_OUTPUT_REVISION)
#error "The header file 'delete.h' doesn't include <QObject>."
#elif Q_MOC_OUTPUT_REVISION != 67
#error "This file was generated using the moc from 5.12.8. It"
#error "cannot be used with the include files from this version of Qt."
#error "(The moc has changed too much.)"
#endif

QT_BEGIN_MOC_NAMESPACE
QT_WARNING_PUSH
QT_WARNING_DISABLE_DEPRECATED
struct qt_meta_stringdata_Delete_t {
    QByteArrayData data[3];
    char stringdata0[15];
};
#define QT_MOC_LITERAL(idx, ofs, len) \
    Q_STATIC_BYTE_ARRAY_DATA_HEADER_INITIALIZER_WITH_OFFSET(len, \
    qptrdiff(offsetof(qt_meta_stringdata_Delete_t, stringdata0) + ofs \
        - idx * sizeof(QByteArrayData)) \
    )
static const qt_meta_stringdata_Delete_t qt_meta_stringdata_Delete = {
    {
QT_MOC_LITERAL(0, 0, 6), // "Delete"
QT_MOC_LITERAL(1, 7, 6), // "accept"
QT_MOC_LITERAL(2, 14, 0) // ""

    },
    "Delete\0accept\0"
};
#undef QT_MOC_LITERAL

static const uint qt_meta_data_Delete[] = {

 // content:
       8,       // revision
       0,       // classname
       0,    0, // classinfo
       1,   14, // methods
       0,    0, // properties
       0,    0, // enums/sets
       0,    0, // constructors
       0,       // flags
       0,       // signalCount

 // slots: name, argc, parameters, tag, flags
       1,    0,   19,    2, 0x0a /* Public */,

 // slots: parameters
    QMetaType::Void,

       0        // eod
};

void Delete::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    if (_c == QMetaObject::InvokeMetaMethod) {
        auto *_t = static_cast<Delete *>(_o);
        Q_UNUSED(_t)
        switch (_id) {
        case 0: _t->accept(); break;
        default: ;
        }
    }
    Q_UNUSED(_a);
}

QT_INIT_METAOBJECT const QMetaObject Delete::staticMetaObject = { {
    &QDialog::staticMetaObject,
    qt_meta_stringdata_Delete.data,
    qt_meta_data_Delete,
    qt_static_metacall,
    nullptr,
    nullptr
} };


const QMetaObject *Delete::metaObject() const
{
    return QObject::d_ptr->metaObject ? QObject::d_ptr->dynamicMetaObject() : &staticMetaObject;
}

void *Delete::qt_metacast(const char *_clname)
{
    if (!_clname) return nullptr;
    if (!strcmp(_clname, qt_meta_stringdata_Delete.stringdata0))
        return static_cast<void*>(this);
    return QDialog::qt_metacast(_clname);
}

int Delete::qt_metacall(QMetaObject::Call _c, int _id, void **_a)
{
    _id = QDialog::qt_metacall(_c, _id, _a);
    if (_id < 0)
        return _id;
    if (_c == QMetaObject::InvokeMetaMethod) {
        if (_id < 1)
            qt_static_metacall(this, _c, _id, _a);
        _id -= 1;
    } else if (_c == QMetaObject::RegisterMethodArgumentMetaType) {
        if (_id < 1)
            *reinterpret_cast<int*>(_a[0]) = -1;
        _id -= 1;
    }
    return _id;
}
QT_WARNING_POP
QT_END_MOC_NAMESPACE
 BIN +1.19 MB 
4mv/build-4mv_curse-Desktop-Debug/moc_delete.o
Binary file not shown.
 94  
4mv/build-4mv_curse-Desktop-Debug/moc_faq.cpp
@@ -0,0 +1,94 @@
/****************************************************************************
** Meta object code from reading C++ file 'faq.h'
**
** Created by: The Qt Meta Object Compiler version 67 (Qt 5.12.8)
**
** WARNING! All changes made in this file will be lost!
*****************************************************************************/

#include "../4mv_curse/faq.h"
#include <QtCore/qbytearray.h>
#include <QtCore/qmetatype.h>
#if !defined(Q_MOC_OUTPUT_REVISION)
#error "The header file 'faq.h' doesn't include <QObject>."
#elif Q_MOC_OUTPUT_REVISION != 67
#error "This file was generated using the moc from 5.12.8. It"
#error "cannot be used with the include files from this version of Qt."
#error "(The moc has changed too much.)"
#endif

QT_BEGIN_MOC_NAMESPACE
QT_WARNING_PUSH
QT_WARNING_DISABLE_DEPRECATED
struct qt_meta_stringdata_FAQ_t {
    QByteArrayData data[1];
    char stringdata0[4];
};
#define QT_MOC_LITERAL(idx, ofs, len) \
    Q_STATIC_BYTE_ARRAY_DATA_HEADER_INITIALIZER_WITH_OFFSET(len, \
    qptrdiff(offsetof(qt_meta_stringdata_FAQ_t, stringdata0) + ofs \
        - idx * sizeof(QByteArrayData)) \
    )
static const qt_meta_stringdata_FAQ_t qt_meta_stringdata_FAQ = {
    {
QT_MOC_LITERAL(0, 0, 3) // "FAQ"

    },
    "FAQ"
};
#undef QT_MOC_LITERAL

static const uint qt_meta_data_FAQ[] = {

 // content:
       8,       // revision
       0,       // classname
       0,    0, // classinfo
       0,    0, // methods
       0,    0, // properties
       0,    0, // enums/sets
       0,    0, // constructors
       0,       // flags
       0,       // signalCount

       0        // eod
};

void FAQ::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    Q_UNUSED(_o);
    Q_UNUSED(_id);
    Q_UNUSED(_c);
    Q_UNUSED(_a);
}

QT_INIT_METAOBJECT const QMetaObject FAQ::staticMetaObject = { {
    &QDialog::staticMetaObject,
    qt_meta_stringdata_FAQ.data,
    qt_meta_data_FAQ,
    qt_static_metacall,
    nullptr,
    nullptr
} };


const QMetaObject *FAQ::metaObject() const
{
    return QObject::d_ptr->metaObject ? QObject::d_ptr->dynamicMetaObject() : &staticMetaObject;
}

void *FAQ::qt_metacast(const char *_clname)
{
    if (!_clname) return nullptr;
    if (!strcmp(_clname, qt_meta_stringdata_FAQ.stringdata0))
        return static_cast<void*>(this);
    return QDialog::qt_metacast(_clname);
}

int FAQ::qt_metacall(QMetaObject::Call _c, int _id, void **_a)
{
    _id = QDialog::qt_metacall(_c, _id, _a);
    return _id;
}
QT_WARNING_POP
QT_END_MOC_NAMESPACE
 BIN +1.19 MB 
4mv/build-4mv_curse-Desktop-Debug/moc_faq.o
Binary file not shown.
 145  
4mv/build-4mv_curse-Desktop-Debug/moc_mainwindow.cpp
@@ -0,0 +1,145 @@
/****************************************************************************
** Meta object code from reading C++ file 'mainwindow.h'
**
** Created by: The Qt Meta Object Compiler version 67 (Qt 5.12.8)
**
** WARNING! All changes made in this file will be lost!
*****************************************************************************/

#include "../4mv_curse/mainwindow.h"
#include <QtCore/qbytearray.h>
#include <QtCore/qmetatype.h>
#if !defined(Q_MOC_OUTPUT_REVISION)
#error "The header file 'mainwindow.h' doesn't include <QObject>."
#elif Q_MOC_OUTPUT_REVISION != 67
#error "This file was generated using the moc from 5.12.8. It"
#error "cannot be used with the include files from this version of Qt."
#error "(The moc has changed too much.)"
#endif

QT_BEGIN_MOC_NAMESPACE
QT_WARNING_PUSH
QT_WARNING_DISABLE_DEPRECATED
struct qt_meta_stringdata_MainWindow_t {
    QByteArrayData data[9];
    char stringdata0[163];
};
#define QT_MOC_LITERAL(idx, ofs, len) \
    Q_STATIC_BYTE_ARRAY_DATA_HEADER_INITIALIZER_WITH_OFFSET(len, \
    qptrdiff(offsetof(qt_meta_stringdata_MainWindow_t, stringdata0) + ofs \
        - idx * sizeof(QByteArrayData)) \
    )
static const qt_meta_stringdata_MainWindow_t qt_meta_stringdata_MainWindow = {
    {
QT_MOC_LITERAL(0, 0, 10), // "MainWindow"
QT_MOC_LITERAL(1, 11, 8), // "slotHelp"
QT_MOC_LITERAL(2, 20, 0), // ""
QT_MOC_LITERAL(3, 21, 21), // "on_pushButton_clicked"
QT_MOC_LITERAL(4, 43, 23), // "on_pushButton_2_clicked"
QT_MOC_LITERAL(5, 67, 23), // "on_pushButton_3_clicked"
QT_MOC_LITERAL(6, 91, 23), // "on_pushButton_4_clicked"
QT_MOC_LITERAL(7, 115, 23), // "on_pushButton_5_clicked"
QT_MOC_LITERAL(8, 139, 23) // "on_pushButton_6_clicked"

    },
    "MainWindow\0slotHelp\0\0on_pushButton_clicked\0"
    "on_pushButton_2_clicked\0on_pushButton_3_clicked\0"
    "on_pushButton_4_clicked\0on_pushButton_5_clicked\0"
    "on_pushButton_6_clicked"
};
#undef QT_MOC_LITERAL

static const uint qt_meta_data_MainWindow[] = {

 // content:
       8,       // revision
       0,       // classname
       0,    0, // classinfo
       7,   14, // methods
       0,    0, // properties
       0,    0, // enums/sets
       0,    0, // constructors
       0,       // flags
       0,       // signalCount

 // slots: name, argc, parameters, tag, flags
       1,    0,   49,    2, 0x08 /* Private */,
       3,    0,   50,    2, 0x08 /* Private */,
       4,    0,   51,    2, 0x08 /* Private */,
       5,    0,   52,    2, 0x08 /* Private */,
       6,    0,   53,    2, 0x08 /* Private */,
       7,    0,   54,    2, 0x08 /* Private */,
       8,    0,   55,    2, 0x08 /* Private */,

 // slots: parameters
    QMetaType::Void,
    QMetaType::Void,
    QMetaType::Void,
    QMetaType::Void,
    QMetaType::Void,
    QMetaType::Void,
    QMetaType::Void,

       0        // eod
};

void MainWindow::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    if (_c == QMetaObject::InvokeMetaMethod) {
        auto *_t = static_cast<MainWindow *>(_o);
        Q_UNUSED(_t)
        switch (_id) {
        case 0: _t->slotHelp(); break;
        case 1: _t->on_pushButton_clicked(); break;
        case 2: _t->on_pushButton_2_clicked(); break;
        case 3: _t->on_pushButton_3_clicked(); break;
        case 4: _t->on_pushButton_4_clicked(); break;
        case 5: _t->on_pushButton_5_clicked(); break;
        case 6: _t->on_pushButton_6_clicked(); break;
        default: ;
        }
    }
    Q_UNUSED(_a);
}

QT_INIT_METAOBJECT const QMetaObject MainWindow::staticMetaObject = { {
    &QMainWindow::staticMetaObject,
    qt_meta_stringdata_MainWindow.data,
    qt_meta_data_MainWindow,
    qt_static_metacall,
    nullptr,
    nullptr
} };


const QMetaObject *MainWindow::metaObject() const
{
    return QObject::d_ptr->metaObject ? QObject::d_ptr->dynamicMetaObject() : &staticMetaObject;
}

void *MainWindow::qt_metacast(const char *_clname)
{
    if (!_clname) return nullptr;
    if (!strcmp(_clname, qt_meta_stringdata_MainWindow.stringdata0))
        return static_cast<void*>(this);
    return QMainWindow::qt_metacast(_clname);
}

int MainWindow::qt_metacall(QMetaObject::Call _c, int _id, void **_a)
{
    _id = QMainWindow::qt_metacall(_c, _id, _a);
    if (_id < 0)
        return _id;
    if (_c == QMetaObject::InvokeMetaMethod) {
        if (_id < 7)
            qt_static_metacall(this, _c, _id, _a);
        _id -= 7;
    } else if (_c == QMetaObject::RegisterMethodArgumentMetaType) {
        if (_id < 7)
            *reinterpret_cast<int*>(_a[0]) = -1;
        _id -= 7;
    }
    return _id;
}
QT_WARNING_POP
QT_END_MOC_NAMESPACE
 BIN +1.19 MB 
4mv/build-4mv_curse-Desktop-Debug/moc_mainwindow.o
Binary file not shown.
 118  
4mv/build-4mv_curse-Desktop-Debug/moc_payment.cpp
@@ -0,0 +1,118 @@
/****************************************************************************
** Meta object code from reading C++ file 'payment.h'
**
** Created by: The Qt Meta Object Compiler version 67 (Qt 5.12.8)
**
** WARNING! All changes made in this file will be lost!
*****************************************************************************/

#include "../4mv_curse/payment.h"
#include <QtCore/qbytearray.h>
#include <QtCore/qmetatype.h>
#if !defined(Q_MOC_OUTPUT_REVISION)
#error "The header file 'payment.h' doesn't include <QObject>."
#elif Q_MOC_OUTPUT_REVISION != 67
#error "This file was generated using the moc from 5.12.8. It"
#error "cannot be used with the include files from this version of Qt."
#error "(The moc has changed too much.)"
#endif

QT_BEGIN_MOC_NAMESPACE
QT_WARNING_PUSH
QT_WARNING_DISABLE_DEPRECATED
struct qt_meta_stringdata_Payment_t {
    QByteArrayData data[3];
    char stringdata0[16];
};
#define QT_MOC_LITERAL(idx, ofs, len) \
    Q_STATIC_BYTE_ARRAY_DATA_HEADER_INITIALIZER_WITH_OFFSET(len, \
    qptrdiff(offsetof(qt_meta_stringdata_Payment_t, stringdata0) + ofs \
        - idx * sizeof(QByteArrayData)) \
    )
static const qt_meta_stringdata_Payment_t qt_meta_stringdata_Payment = {
    {
QT_MOC_LITERAL(0, 0, 7), // "Payment"
QT_MOC_LITERAL(1, 8, 6), // "accept"
QT_MOC_LITERAL(2, 15, 0) // ""

    },
    "Payment\0accept\0"
};
#undef QT_MOC_LITERAL

static const uint qt_meta_data_Payment[] = {

 // content:
       8,       // revision
       0,       // classname
       0,    0, // classinfo
       1,   14, // methods
       0,    0, // properties
       0,    0, // enums/sets
       0,    0, // constructors
       0,       // flags
       0,       // signalCount

 // slots: name, argc, parameters, tag, flags
       1,    0,   19,    2, 0x0a /* Public */,

 // slots: parameters
    QMetaType::Void,

       0        // eod
};

void Payment::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    if (_c == QMetaObject::InvokeMetaMethod) {
        auto *_t = static_cast<Payment *>(_o);
        Q_UNUSED(_t)
        switch (_id) {
        case 0: _t->accept(); break;
        default: ;
        }
    }
    Q_UNUSED(_a);
}

QT_INIT_METAOBJECT const QMetaObject Payment::staticMetaObject = { {
    &QDialog::staticMetaObject,
    qt_meta_stringdata_Payment.data,
    qt_meta_data_Payment,
    qt_static_metacall,
    nullptr,
    nullptr
} };


const QMetaObject *Payment::metaObject() const
{
    return QObject::d_ptr->metaObject ? QObject::d_ptr->dynamicMetaObject() : &staticMetaObject;
}

void *Payment::qt_metacast(const char *_clname)
{
    if (!_clname) return nullptr;
    if (!strcmp(_clname, qt_meta_stringdata_Payment.stringdata0))
        return static_cast<void*>(this);
    return QDialog::qt_metacast(_clname);
}

int Payment::qt_metacall(QMetaObject::Call _c, int _id, void **_a)
{
    _id = QDialog::qt_metacall(_c, _id, _a);
    if (_id < 0)
        return _id;
    if (_c == QMetaObject::InvokeMetaMethod) {
        if (_id < 1)
            qt_static_metacall(this, _c, _id, _a);
        _id -= 1;
    } else if (_c == QMetaObject::RegisterMethodArgumentMetaType) {
        if (_id < 1)
            *reinterpret_cast<int*>(_a[0]) = -1;
        _id -= 1;
    }
    return _id;
}
QT_WARNING_POP
QT_END_MOC_NAMESPACE
 BIN +1.19 MB 
4mv/build-4mv_curse-Desktop-Debug/moc_payment.o
Binary file not shown.
 390  
4mv/build-4mv_curse-Desktop-Debug/moc_predefs.h
@@ -0,0 +1,390 @@
#define __SSP_STRONG__ 3
#define __DBL_MIN_EXP__ (-1021)
#define __FLT32X_MAX_EXP__ 1024
#define __cpp_attributes 200809
#define __UINT_LEAST16_MAX__ 0xffff
#define __ATOMIC_ACQUIRE 2
#define __FLT128_MAX_10_EXP__ 4932
#define __FLT_MIN__ 1.17549435082228750796873653722224568e-38F
#define __GCC_IEC_559_COMPLEX 2
#define __cpp_aggregate_nsdmi 201304
#define __UINT_LEAST8_TYPE__ unsigned char
#define __SIZEOF_FLOAT80__ 16
#define __INTMAX_C(c) c ## L
#define __CHAR_BIT__ 8
#define __UINT8_MAX__ 0xff
#define __WINT_MAX__ 0xffffffffU
#define __FLT32_MIN_EXP__ (-125)
#define __cpp_static_assert 200410
#define __ORDER_LITTLE_ENDIAN__ 1234
#define __SIZE_MAX__ 0xffffffffffffffffUL
#define __WCHAR_MAX__ 0x7fffffff
#define __GCC_HAVE_SYNC_COMPARE_AND_SWAP_1 1
#define __GCC_HAVE_SYNC_COMPARE_AND_SWAP_2 1
#define __GCC_HAVE_SYNC_COMPARE_AND_SWAP_4 1
#define __DBL_DENORM_MIN__ double(4.94065645841246544176568792868221372e-324L)
#define __GCC_HAVE_SYNC_COMPARE_AND_SWAP_8 1
#define __GCC_ATOMIC_CHAR_LOCK_FREE 2
#define __GCC_IEC_559 2
#define __FLT32X_DECIMAL_DIG__ 17
#define __FLT_EVAL_METHOD__ 0
#define __unix__ 1
#define __cpp_binary_literals 201304
#define __FLT64_DECIMAL_DIG__ 17
#define __CET__ 3
#define __GCC_ATOMIC_CHAR32_T_LOCK_FREE 2
#define __x86_64 1
#define __cpp_variadic_templates 200704
#define __UINT_FAST64_MAX__ 0xffffffffffffffffUL
#define __SIG_ATOMIC_TYPE__ int
#define __DBL_MIN_10_EXP__ (-307)
#define __FINITE_MATH_ONLY__ 0
#define __cpp_variable_templates 201304
#define __GNUC_PATCHLEVEL__ 0
#define __FLT32_HAS_DENORM__ 1
#define __UINT_FAST8_MAX__ 0xff
#define __cpp_rvalue_reference 200610
#define __has_include(STR) __has_include__(STR)
#define __DEC64_MAX_EXP__ 385
#define __INT8_C(c) c
#define __INT_LEAST8_WIDTH__ 8
#define __UINT_LEAST64_MAX__ 0xffffffffffffffffUL
#define __SHRT_MAX__ 0x7fff
#define __LDBL_MAX__ 1.18973149535723176502126385303097021e+4932L
#define __FLT64X_MAX_10_EXP__ 4932
#define __UINT_LEAST8_MAX__ 0xff
#define __GCC_ATOMIC_BOOL_LOCK_FREE 2
#define __FLT128_DENORM_MIN__ 6.47517511943802511092443895822764655e-4966F128
#define __UINTMAX_TYPE__ long unsigned int
#define __linux 1
#define __DEC32_EPSILON__ 1E-6DF
#define __FLT_EVAL_METHOD_TS_18661_3__ 0
#define __unix 1
#define __UINT32_MAX__ 0xffffffffU
#define __GXX_EXPERIMENTAL_CXX0X__ 1
#define __LDBL_MAX_EXP__ 16384
#define __FLT128_MIN_EXP__ (-16381)
#define __WINT_MIN__ 0U
#define __linux__ 1
#define __FLT128_MIN_10_EXP__ (-4931)
#define __INT_LEAST16_WIDTH__ 16
#define __SCHAR_MAX__ 0x7f
#define __FLT128_MANT_DIG__ 113
#define __WCHAR_MIN__ (-__WCHAR_MAX__ - 1)
#define __INT64_C(c) c ## L
#define __DBL_DIG__ 15
#define __GCC_ATOMIC_POINTER_LOCK_FREE 2
#define __FLT64X_MANT_DIG__ 64
#define __SIZEOF_INT__ 4
#define __SIZEOF_POINTER__ 8
#define __GCC_ATOMIC_CHAR16_T_LOCK_FREE 2
#define __USER_LABEL_PREFIX__ 
#define __FLT64X_EPSILON__ 1.08420217248550443400745280086994171e-19F64x
#define __STDC_HOSTED__ 1
#define __LDBL_HAS_INFINITY__ 1
#define __FLT32_DIG__ 6
#define __FLT_EPSILON__ 1.19209289550781250000000000000000000e-7F
#define __GXX_WEAK__ 1
#define __SHRT_WIDTH__ 16
#define __LDBL_MIN__ 3.36210314311209350626267781732175260e-4932L
#define __DEC32_MAX__ 9.999999E96DF
#define __cpp_threadsafe_static_init 200806
#define __FLT64X_DENORM_MIN__ 3.64519953188247460252840593361941982e-4951F64x
#define __FLT32X_HAS_INFINITY__ 1
#define __INT32_MAX__ 0x7fffffff
#define __INT_WIDTH__ 32
#define __SIZEOF_LONG__ 8
#define __STDC_IEC_559__ 1
#define __STDC_ISO_10646__ 201706L
#define __UINT16_C(c) c
#define __PTRDIFF_WIDTH__ 64
#define __DECIMAL_DIG__ 21
#define __FLT64_EPSILON__ 2.22044604925031308084726333618164062e-16F64
#define __gnu_linux__ 1
#define __INTMAX_WIDTH__ 64
#define __FLT64_MIN_EXP__ (-1021)
#define __has_include_next(STR) __has_include_next__(STR)
#define __FLT64X_MIN_10_EXP__ (-4931)
#define __LDBL_HAS_QUIET_NAN__ 1
#define __FLT64_MANT_DIG__ 53
#define __GNUC__ 9
#define __GXX_RTTI 1
#define __pie__ 2
#define __MMX__ 1
#define __cpp_delegating_constructors 200604
#define __FLT_HAS_DENORM__ 1
#define __SIZEOF_LONG_DOUBLE__ 16
#define __BIGGEST_ALIGNMENT__ 16
#define __STDC_UTF_16__ 1
#define __FLT64_MAX_10_EXP__ 308
#define __FLT32_HAS_INFINITY__ 1
#define __DBL_MAX__ double(1.79769313486231570814527423731704357e+308L)
#define __cpp_raw_strings 200710
#define __INT_FAST32_MAX__ 0x7fffffffffffffffL
#define __DBL_HAS_INFINITY__ 1
#define __HAVE_SPECULATION_SAFE_VALUE 1
#define __DEC32_MIN_EXP__ (-94)
#define __INTPTR_WIDTH__ 64
#define __FLT32X_HAS_DENORM__ 1
#define __INT_FAST16_TYPE__ long int
#define __LDBL_HAS_DENORM__ 1
#define __cplusplus 201402L
#define __cpp_ref_qualifiers 200710
#define __DEC128_MAX__ 9.999999999999999999999999999999999E6144DL
#define __INT_LEAST32_MAX__ 0x7fffffff
#define __DEC32_MIN__ 1E-95DF
#define __DEPRECATED 1
#define __cpp_rvalue_references 200610
#define __DBL_MAX_EXP__ 1024
#define __WCHAR_WIDTH__ 32
#define __FLT32_MAX__ 3.40282346638528859811704183484516925e+38F32
#define __DEC128_EPSILON__ 1E-33DL
#define __SSE2_MATH__ 1
#define __ATOMIC_HLE_RELEASE 131072
#define __PTRDIFF_MAX__ 0x7fffffffffffffffL
#define __amd64 1
#define __ATOMIC_HLE_ACQUIRE 65536
#define __FLT32_HAS_QUIET_NAN__ 1
#define __GNUG__ 9
#define __LONG_LONG_MAX__ 0x7fffffffffffffffLL
#define __SIZEOF_SIZE_T__ 8
#define __cpp_nsdmi 200809
#define __FLT64X_MIN_EXP__ (-16381)
#define __SIZEOF_WINT_T__ 4
#define __LONG_LONG_WIDTH__ 64
#define __cpp_initializer_lists 200806
#define __FLT32_MAX_EXP__ 128
#define __cpp_hex_float 201603
#define __GCC_HAVE_DWARF2_CFI_ASM 1
#define __GXX_ABI_VERSION 1013
#define __FLT128_HAS_INFINITY__ 1
#define __FLT_MIN_EXP__ (-125)
#define __cpp_lambdas 200907
#define __FLT64X_HAS_QUIET_NAN__ 1
#define __INT_FAST64_TYPE__ long int
#define __FLT64_DENORM_MIN__ 4.94065645841246544176568792868221372e-324F64
#define __DBL_MIN__ double(2.22507385850720138309023271733240406e-308L)
#define __PIE__ 2
#define __LP64__ 1
#define __FLT32X_EPSILON__ 2.22044604925031308084726333618164062e-16F32x
#define __DECIMAL_BID_FORMAT__ 1
#define __FLT64_MIN_10_EXP__ (-307)
#define __FLT64X_DECIMAL_DIG__ 21
#define __DEC128_MIN__ 1E-6143DL
#define __REGISTER_PREFIX__ 
#define __UINT16_MAX__ 0xffff
#define __FLT32_MIN__ 1.17549435082228750796873653722224568e-38F32
#define __UINT8_TYPE__ unsigned char
#define __NO_INLINE__ 1
#define __FLT_MANT_DIG__ 24
#define __LDBL_DECIMAL_DIG__ 21
#define __VERSION__ "9.3.0"
#define __UINT64_C(c) c ## UL
#define __cpp_unicode_characters 200704
#define _STDC_PREDEF_H 1
#define __cpp_decltype_auto 201304
#define __GCC_ATOMIC_INT_LOCK_FREE 2
#define __FLT128_MAX_EXP__ 16384
#define __FLT32_MANT_DIG__ 24
#define __FLOAT_WORD_ORDER__ __ORDER_LITTLE_ENDIAN__
#define __STDC_IEC_559_COMPLEX__ 1
#define __FLT128_HAS_DENORM__ 1
#define __FLT128_DIG__ 33
#define __SCHAR_WIDTH__ 8
#define __INT32_C(c) c
#define __DEC64_EPSILON__ 1E-15DD
#define __ORDER_PDP_ENDIAN__ 3412
#define __DEC128_MIN_EXP__ (-6142)
#define __FLT32_MAX_10_EXP__ 38
#define __INT_FAST32_TYPE__ long int
#define __UINT_LEAST16_TYPE__ short unsigned int
#define __FLT64X_HAS_INFINITY__ 1
#define unix 1
#define __DBL_HAS_DENORM__ 1
#define __INT16_MAX__ 0x7fff
#define __cpp_rtti 199711
#define __SIZE_TYPE__ long unsigned int
#define __UINT64_MAX__ 0xffffffffffffffffUL
#define __FLT64X_DIG__ 18
#define __INT8_TYPE__ signed char
#define __cpp_digit_separators 201309
#define __ELF__ 1
#define __GCC_ASM_FLAG_OUTPUTS__ 1
#define __FLT_RADIX__ 2
#define __INT_LEAST16_TYPE__ short int
#define __LDBL_EPSILON__ 1.08420217248550443400745280086994171e-19L
#define __UINTMAX_C(c) c ## UL
#define __GLIBCXX_BITSIZE_INT_N_0 128
#define __k8 1
#define __SIG_ATOMIC_MAX__ 0x7fffffff
#define __GCC_ATOMIC_WCHAR_T_LOCK_FREE 2
#define __SIZEOF_PTRDIFF_T__ 8
#define __FLT32X_MANT_DIG__ 53
#define __x86_64__ 1
#define __FLT32X_MIN_EXP__ (-1021)
#define __DEC32_SUBNORMAL_MIN__ 0.000001E-95DF
#define __INT_FAST16_MAX__ 0x7fffffffffffffffL
#define __FLT64_DIG__ 15
#define __UINT_FAST32_MAX__ 0xffffffffffffffffUL
#define __UINT_LEAST64_TYPE__ long unsigned int
#define __FLT_HAS_QUIET_NAN__ 1
#define __FLT_MAX_10_EXP__ 38
#define __LONG_MAX__ 0x7fffffffffffffffL
#define __FLT64X_HAS_DENORM__ 1
#define __DEC128_SUBNORMAL_MIN__ 0.000000000000000000000000000000001E-6143DL
#define __FLT_HAS_INFINITY__ 1
#define __cpp_unicode_literals 200710
#define __UINT_FAST16_TYPE__ long unsigned int
#define __DEC64_MAX__ 9.999999999999999E384DD
#define __INT_FAST32_WIDTH__ 64
#define __CHAR16_TYPE__ short unsigned int
#define __PRAGMA_REDEFINE_EXTNAME 1
#define __SIZE_WIDTH__ 64
#define __SEG_FS 1
#define __INT_LEAST16_MAX__ 0x7fff
#define __DEC64_MANT_DIG__ 16
#define __INT64_MAX__ 0x7fffffffffffffffL
#define __UINT_LEAST32_MAX__ 0xffffffffU
#define __SEG_GS 1
#define __FLT32_DENORM_MIN__ 1.40129846432481707092372958328991613e-45F32
#define __GCC_ATOMIC_LONG_LOCK_FREE 2
#define __SIG_ATOMIC_WIDTH__ 32
#define __INT_LEAST64_TYPE__ long int
#define __INT16_TYPE__ short int
#define __INT_LEAST8_TYPE__ signed char
#define __DEC32_MAX_EXP__ 97
#define __INT_FAST8_MAX__ 0x7f
#define __FLT128_MAX__ 1.18973149535723176508575932662800702e+4932F128
#define __INTPTR_MAX__ 0x7fffffffffffffffL
#define __cpp_sized_deallocation 201309
#define linux 1
#define __cpp_range_based_for 200907
#define __FLT64_HAS_QUIET_NAN__ 1
#define __FLT32_MIN_10_EXP__ (-37)
#define __SSE2__ 1
#define __EXCEPTIONS 1
#define __LDBL_MANT_DIG__ 64
#define __DBL_HAS_QUIET_NAN__ 1
#define __FLT64_HAS_INFINITY__ 1
#define __FLT64X_MAX__ 1.18973149535723176502126385303097021e+4932F64x
#define __SIG_ATOMIC_MIN__ (-__SIG_ATOMIC_MAX__ - 1)
#define __code_model_small__ 1
#define __cpp_return_type_deduction 201304
#define __k8__ 1
#define __INTPTR_TYPE__ long int
#define __UINT16_TYPE__ short unsigned int
#define __WCHAR_TYPE__ int
#define __SIZEOF_FLOAT__ 4
#define __pic__ 2
#define __UINTPTR_MAX__ 0xffffffffffffffffUL
#define __INT_FAST64_WIDTH__ 64
#define __DEC64_MIN_EXP__ (-382)
#define __cpp_decltype 200707
#define __FLT32_DECIMAL_DIG__ 9
#define __INT_FAST64_MAX__ 0x7fffffffffffffffL
#define __GCC_ATOMIC_TEST_AND_SET_TRUEVAL 1
#define __FLT_DIG__ 6
#define __FLT64X_MAX_EXP__ 16384
#define __UINT_FAST64_TYPE__ long unsigned int
#define __INT_MAX__ 0x7fffffff
#define __amd64__ 1
#define __INT64_TYPE__ long int
#define __FLT_MAX_EXP__ 128
#define __ORDER_BIG_ENDIAN__ 4321
#define __DBL_MANT_DIG__ 53
#define __cpp_inheriting_constructors 201511
#define __SIZEOF_FLOAT128__ 16
#define __INT_LEAST64_MAX__ 0x7fffffffffffffffL
#define __DEC64_MIN__ 1E-383DD
#define __WINT_TYPE__ unsigned int
#define __UINT_LEAST32_TYPE__ unsigned int
#define __SIZEOF_SHORT__ 2
#define __SSE__ 1
#define __LDBL_MIN_EXP__ (-16381)
#define __FLT64_MAX__ 1.79769313486231570814527423731704357e+308F64
#define __WINT_WIDTH__ 32
#define __INT_LEAST8_MAX__ 0x7f
#define __FLT32X_MAX_10_EXP__ 308
#define __SIZEOF_INT128__ 16
#define __LDBL_MAX_10_EXP__ 4932
#define __ATOMIC_RELAXED 0
#define __DBL_EPSILON__ double(2.22044604925031308084726333618164062e-16L)
#define __FLT128_MIN__ 3.36210314311209350626267781732175260e-4932F128
#define _LP64 1
#define __UINT8_C(c) c
#define __FLT64_MAX_EXP__ 1024
#define __INT_LEAST32_TYPE__ int
#define __SIZEOF_WCHAR_T__ 4
#define __FLT128_HAS_QUIET_NAN__ 1
#define __INT_FAST8_TYPE__ signed char
#define __FLT64X_MIN__ 3.36210314311209350626267781732175260e-4932F64x
#define __GNUC_STDC_INLINE__ 1
#define __FLT64_HAS_DENORM__ 1
#define __FLT32_EPSILON__ 1.19209289550781250000000000000000000e-7F32
#define __DBL_DECIMAL_DIG__ 17
#define __STDC_UTF_32__ 1
#define __INT_FAST8_WIDTH__ 8
#define __FXSR__ 1
#define __DEC_EVAL_METHOD__ 2
#define __FLT32X_MAX__ 1.79769313486231570814527423731704357e+308F32x
#define __cpp_runtime_arrays 198712
#define __UINT64_TYPE__ long unsigned int
#define __UINT32_C(c) c ## U
#define __INTMAX_MAX__ 0x7fffffffffffffffL
#define __cpp_alias_templates 200704
#define __BYTE_ORDER__ __ORDER_LITTLE_ENDIAN__
#define __FLT_DENORM_MIN__ 1.40129846432481707092372958328991613e-45F
#define __INT8_MAX__ 0x7f
#define __LONG_WIDTH__ 64
#define __PIC__ 2
#define __UINT_FAST32_TYPE__ long unsigned int
#define __CHAR32_TYPE__ unsigned int
#define __FLT_MAX__ 3.40282346638528859811704183484516925e+38F
#define __cpp_constexpr 201304
#define __INT32_TYPE__ int
#define __SIZEOF_DOUBLE__ 8
#define __cpp_exceptions 199711
#define __FLT_MIN_10_EXP__ (-37)
#define __FLT64_MIN__ 2.22507385850720138309023271733240406e-308F64
#define __INT_LEAST32_WIDTH__ 32
#define __INTMAX_TYPE__ long int
#define __DEC128_MAX_EXP__ 6145
#define __FLT32X_HAS_QUIET_NAN__ 1
#define __ATOMIC_CONSUME 1
#define __GNUC_MINOR__ 3
#define __GLIBCXX_TYPE_INT_N_0 __int128
#define __INT_FAST16_WIDTH__ 64
#define __UINTMAX_MAX__ 0xffffffffffffffffUL
#define __DEC32_MANT_DIG__ 7
#define __FLT32X_DENORM_MIN__ 4.94065645841246544176568792868221372e-324F32x
#define __DBL_MAX_10_EXP__ 308
#define __LDBL_DENORM_MIN__ 3.64519953188247460252840593361941982e-4951L
#define __INT16_C(c) c
#define __cpp_generic_lambdas 201304
#define __STDC__ 1
#define __FLT32X_DIG__ 15
#define __PTRDIFF_TYPE__ long int
#define __ATOMIC_SEQ_CST 5
#define __UINT32_TYPE__ unsigned int
#define __FLT32X_MIN_10_EXP__ (-307)
#define __UINTPTR_TYPE__ long unsigned int
#define __DEC64_SUBNORMAL_MIN__ 0.000000000000001E-383DD
#define __DEC128_MANT_DIG__ 34
#define __LDBL_MIN_10_EXP__ (-4931)
#define __FLT128_EPSILON__ 1.92592994438723585305597794258492732e-34F128
#define __SSE_MATH__ 1
#define __SIZEOF_LONG_LONG__ 8
#define __cpp_user_defined_literals 200809
#define __FLT128_DECIMAL_DIG__ 36
#define __GCC_ATOMIC_LLONG_LOCK_FREE 2
#define __FLT32X_MIN__ 2.22507385850720138309023271733240406e-308F32x
#define __LDBL_DIG__ 18
#define __FLT_DECIMAL_DIG__ 9
#define __UINT_FAST16_MAX__ 0xffffffffffffffffUL
#define __GCC_ATOMIC_SHORT_LOCK_FREE 2
#define __INT_LEAST64_WIDTH__ 64
#define __UINT_FAST8_TYPE__ unsigned char
#define _GNU_SOURCE 1
#define __cpp_init_captures 201304
#define __ATOMIC_ACQ_REL 4
#define __ATOMIC_RELEASE 3
 118  
4mv/build-4mv_curse-Desktop-Debug/moc_update.cpp
@@ -0,0 +1,118 @@
/****************************************************************************
** Meta object code from reading C++ file 'update.h'
**
** Created by: The Qt Meta Object Compiler version 67 (Qt 5.12.8)
**
** WARNING! All changes made in this file will be lost!
*****************************************************************************/

#include "../4mv_curse/update.h"
#include <QtCore/qbytearray.h>
#include <QtCore/qmetatype.h>
#if !defined(Q_MOC_OUTPUT_REVISION)
#error "The header file 'update.h' doesn't include <QObject>."
#elif Q_MOC_OUTPUT_REVISION != 67
#error "This file was generated using the moc from 5.12.8. It"
#error "cannot be used with the include files from this version of Qt."
#error "(The moc has changed too much.)"
#endif

QT_BEGIN_MOC_NAMESPACE
QT_WARNING_PUSH
QT_WARNING_DISABLE_DEPRECATED
struct qt_meta_stringdata_Update_t {
    QByteArrayData data[3];
    char stringdata0[15];
};
#define QT_MOC_LITERAL(idx, ofs, len) \
    Q_STATIC_BYTE_ARRAY_DATA_HEADER_INITIALIZER_WITH_OFFSET(len, \
    qptrdiff(offsetof(qt_meta_stringdata_Update_t, stringdata0) + ofs \
        - idx * sizeof(QByteArrayData)) \
    )
static const qt_meta_stringdata_Update_t qt_meta_stringdata_Update = {
    {
QT_MOC_LITERAL(0, 0, 6), // "Update"
QT_MOC_LITERAL(1, 7, 6), // "accept"
QT_MOC_LITERAL(2, 14, 0) // ""

    },
    "Update\0accept\0"
};
#undef QT_MOC_LITERAL

static const uint qt_meta_data_Update[] = {

 // content:
       8,       // revision
       0,       // classname
       0,    0, // classinfo
       1,   14, // methods
       0,    0, // properties
       0,    0, // enums/sets
       0,    0, // constructors
       0,       // flags
       0,       // signalCount

 // slots: name, argc, parameters, tag, flags
       1,    0,   19,    2, 0x0a /* Public */,

 // slots: parameters
    QMetaType::Void,

       0        // eod
};

void Update::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    if (_c == QMetaObject::InvokeMetaMethod) {
        auto *_t = static_cast<Update *>(_o);
        Q_UNUSED(_t)
        switch (_id) {
        case 0: _t->accept(); break;
        default: ;
        }
    }
    Q_UNUSED(_a);
}

QT_INIT_METAOBJECT const QMetaObject Update::staticMetaObject = { {
    &QDialog::staticMetaObject,
    qt_meta_stringdata_Update.data,
    qt_meta_data_Update,
    qt_static_metacall,
    nullptr,
    nullptr
} };


const QMetaObject *Update::metaObject() const
{
    return QObject::d_ptr->metaObject ? QObject::d_ptr->dynamicMetaObject() : &staticMetaObject;
}

void *Update::qt_metacast(const char *_clname)
{
    if (!_clname) return nullptr;
    if (!strcmp(_clname, qt_meta_stringdata_Update.stringdata0))
        return static_cast<void*>(this);
    return QDialog::qt_metacast(_clname);
}

int Update::qt_metacall(QMetaObject::Call _c, int _id, void **_a)
{
    _id = QDialog::qt_metacall(_c, _id, _a);
    if (_id < 0)
        return _id;
    if (_c == QMetaObject::InvokeMetaMethod) {
        if (_id < 1)
            qt_static_metacall(this, _c, _id, _a);
        _id -= 1;
    } else if (_c == QMetaObject::RegisterMethodArgumentMetaType) {
        if (_id < 1)
            *reinterpret_cast<int*>(_a[0]) = -1;
        _id -= 1;
    }
    return _id;
}
QT_WARNING_POP
QT_END_MOC_NAMESPACE
 BIN +1.19 MB 
4mv/build-4mv_curse-Desktop-Debug/moc_update.o
Binary file not shown.
 BIN +1.26 MB 
4mv/build-4mv_curse-Desktop-Debug/payment.o
Binary file not shown.
 530  
4mv/build-4mv_curse-Desktop-Debug/qrc_help.cpp
Large diffs are not rendered by default.

 BIN +13.3 KB 
4mv/build-4mv_curse-Desktop-Debug/qrc_help.o
Binary file not shown.
 93  
4mv/build-4mv_curse-Desktop-Debug/ui_add.h
@@ -0,0 +1,93 @@
/********************************************************************************
** Form generated from reading UI file 'add.ui'
**
** Created by: Qt User Interface Compiler version 5.12.8
**
** WARNING! All changes made in this file will be lost when recompiling UI file!
********************************************************************************/

#ifndef UI_ADD_H
#define UI_ADD_H

#include <QtCore/QVariant>
#include <QtWidgets/QApplication>
#include <QtWidgets/QDialog>
#include <QtWidgets/QDialogButtonBox>
#include <QtWidgets/QLabel>
#include <QtWidgets/QTextEdit>

QT_BEGIN_NAMESPACE

class Ui_Add
{
public:
    QDialogButtonBox *buttonBox;
    QLabel *label;
    QLabel *label_2;
    QLabel *label_3;
    QLabel *label_4;
    QTextEdit *textEdit;
    QTextEdit *textEdit_2;
    QTextEdit *textEdit_3;
    QTextEdit *textEdit_4;

    void setupUi(QDialog *Add)
    {
        if (Add->objectName().isEmpty())
            Add->setObjectName(QString::fromUtf8("Add"));
        Add->resize(368, 208);
        buttonBox = new QDialogButtonBox(Add);
        buttonBox->setObjectName(QString::fromUtf8("buttonBox"));
        buttonBox->setGeometry(QRect(0, 170, 331, 32));
        buttonBox->setOrientation(Qt::Horizontal);
        buttonBox->setStandardButtons(QDialogButtonBox::Cancel|QDialogButtonBox::Ok);
        label = new QLabel(Add);
        label->setObjectName(QString::fromUtf8("label"));
        label->setGeometry(QRect(10, 10, 67, 31));
        label_2 = new QLabel(Add);
        label_2->setObjectName(QString::fromUtf8("label_2"));
        label_2->setGeometry(QRect(10, 50, 67, 31));
        label_3 = new QLabel(Add);
        label_3->setObjectName(QString::fromUtf8("label_3"));
        label_3->setGeometry(QRect(10, 90, 67, 31));
        label_4 = new QLabel(Add);
        label_4->setObjectName(QString::fromUtf8("label_4"));
        label_4->setGeometry(QRect(10, 130, 141, 31));
        textEdit = new QTextEdit(Add);
        textEdit->setObjectName(QString::fromUtf8("textEdit"));
        textEdit->setGeometry(QRect(160, 10, 201, 31));
        textEdit_2 = new QTextEdit(Add);
        textEdit_2->setObjectName(QString::fromUtf8("textEdit_2"));
        textEdit_2->setGeometry(QRect(160, 50, 201, 31));
        textEdit_3 = new QTextEdit(Add);
        textEdit_3->setObjectName(QString::fromUtf8("textEdit_3"));
        textEdit_3->setGeometry(QRect(160, 90, 201, 31));
        textEdit_4 = new QTextEdit(Add);
        textEdit_4->setObjectName(QString::fromUtf8("textEdit_4"));
        textEdit_4->setGeometry(QRect(160, 130, 201, 31));

        retranslateUi(Add);
        QObject::connect(buttonBox, SIGNAL(accepted()), Add, SLOT(accept()));
        QObject::connect(buttonBox, SIGNAL(rejected()), Add, SLOT(reject()));

        QMetaObject::connectSlotsByName(Add);
    } // setupUi

    void retranslateUi(QDialog *Add)
    {
        Add->setWindowTitle(QApplication::translate("Add", "Add", nullptr));
        label->setText(QApplication::translate("Add", "\320\244\320\230\320\236", nullptr));
        label_2->setText(QApplication::translate("Add", "\320\277\320\276\320\273", nullptr));
        label_3->setText(QApplication::translate("Add", "\321\202\320\265\320\273\320\265\321\204\320\276\320\275", nullptr));
        label_4->setText(QApplication::translate("Add", "\320\264\320\260\321\202\320\260 \321\200\320\265\320\263\320\270\321\201\321\202\321\200\320\260\321\206\320\270\320\270", nullptr));
    } // retranslateUi

};

namespace Ui {
    class Add: public Ui_Add {};
} // namespace Ui

QT_END_NAMESPACE

#endif // UI_ADD_H
 66  
4mv/build-4mv_curse-Desktop-Debug/ui_delete.h
@@ -0,0 +1,66 @@
/********************************************************************************
** Form generated from reading UI file 'delete.ui'
**
** Created by: Qt User Interface Compiler version 5.12.8
**
** WARNING! All changes made in this file will be lost when recompiling UI file!
********************************************************************************/

#ifndef UI_DELETE_H
#define UI_DELETE_H

#include <QtCore/QVariant>
#include <QtWidgets/QApplication>
#include <QtWidgets/QDialog>
#include <QtWidgets/QDialogButtonBox>
#include <QtWidgets/QLabel>
#include <QtWidgets/QTextEdit>

QT_BEGIN_NAMESPACE

class Ui_Delete
{
public:
    QDialogButtonBox *buttonBox;
    QLabel *label;
    QTextEdit *textEdit;

    void setupUi(QDialog *Delete)
    {
        if (Delete->objectName().isEmpty())
            Delete->setObjectName(QString::fromUtf8("Delete"));
        Delete->resize(268, 93);
        buttonBox = new QDialogButtonBox(Delete);
        buttonBox->setObjectName(QString::fromUtf8("buttonBox"));
        buttonBox->setGeometry(QRect(10, 50, 231, 32));
        buttonBox->setOrientation(Qt::Horizontal);
        buttonBox->setStandardButtons(QDialogButtonBox::Cancel|QDialogButtonBox::Ok);
        label = new QLabel(Delete);
        label->setObjectName(QString::fromUtf8("label"));
        label->setGeometry(QRect(10, 10, 67, 31));
        textEdit = new QTextEdit(Delete);
        textEdit->setObjectName(QString::fromUtf8("textEdit"));
        textEdit->setGeometry(QRect(90, 10, 161, 31));

        retranslateUi(Delete);
        QObject::connect(buttonBox, SIGNAL(accepted()), Delete, SLOT(accept()));
        QObject::connect(buttonBox, SIGNAL(rejected()), Delete, SLOT(reject()));

        QMetaObject::connectSlotsByName(Delete);
    } // setupUi

    void retranslateUi(QDialog *Delete)
    {
        Delete->setWindowTitle(QApplication::translate("Delete", "Delete", nullptr));
        label->setText(QApplication::translate("Delete", "\320\275\320\276\320\274\320\265\321\200", nullptr));
    } // retranslateUi

};

namespace Ui {
    class Delete: public Ui_Delete {};
} // namespace Ui

QT_END_NAMESPACE

#endif // UI_DELETE_H
 68  
4mv/build-4mv_curse-Desktop-Debug/ui_faq.h
@@ -0,0 +1,68 @@
/********************************************************************************
** Form generated from reading UI file 'faq.ui'
**
** Created by: Qt User Interface Compiler version 5.12.8
**
** WARNING! All changes made in this file will be lost when recompiling UI file!
********************************************************************************/

#ifndef UI_FAQ_H
#define UI_FAQ_H

#include <QtCore/QVariant>
#include <QtWidgets/QApplication>
#include <QtWidgets/QDialog>
#include <QtWidgets/QPushButton>
#include <QtWidgets/QTextBrowser>

QT_BEGIN_NAMESPACE

class Ui_FAQ
{
public:
    QTextBrowser *textBrowser;
    QPushButton *pushButton;
    QPushButton *pushButton_2;
    QPushButton *pushButton_3;

    void setupUi(QDialog *FAQ)
    {
        if (FAQ->objectName().isEmpty())
            FAQ->setObjectName(QString::fromUtf8("FAQ"));
        FAQ->resize(512, 394);
        textBrowser = new QTextBrowser(FAQ);
        textBrowser->setObjectName(QString::fromUtf8("textBrowser"));
        textBrowser->setGeometry(QRect(10, 10, 491, 331));
        textBrowser->setStyleSheet(QString::fromUtf8("background-color: rgb(114, 159, 207);"));
        pushButton = new QPushButton(FAQ);
        pushButton->setObjectName(QString::fromUtf8("pushButton"));
        pushButton->setGeometry(QRect(20, 350, 89, 25));
        pushButton_2 = new QPushButton(FAQ);
        pushButton_2->setObjectName(QString::fromUtf8("pushButton_2"));
        pushButton_2->setGeometry(QRect(250, 350, 89, 25));
        pushButton_3 = new QPushButton(FAQ);
        pushButton_3->setObjectName(QString::fromUtf8("pushButton_3"));
        pushButton_3->setGeometry(QRect(380, 350, 89, 25));

        retranslateUi(FAQ);

        QMetaObject::connectSlotsByName(FAQ);
    } // setupUi

    void retranslateUi(QDialog *FAQ)
    {
        FAQ->setWindowTitle(QApplication::translate("FAQ", "FAQ", nullptr));
        pushButton->setText(QApplication::translate("FAQ", "\320\264\320\276\320\274\320\276\320\271", nullptr));
        pushButton_2->setText(QApplication::translate("FAQ", "<<", nullptr));
        pushButton_3->setText(QApplication::translate("FAQ", ">>", nullptr));
    } // retranslateUi

};

namespace Ui {
    class FAQ: public Ui_FAQ {};
} // namespace Ui

QT_END_NAMESPACE

#endif // UI_FAQ_H
 221  
4mv/build-4mv_curse-Desktop-Debug/ui_mainwindow.h
@@ -0,0 +1,221 @@
/********************************************************************************
** Form generated from reading UI file 'mainwindow.ui'
**
** Created by: Qt User Interface Compiler version 5.12.8
**
** WARNING! All changes made in this file will be lost when recompiling UI file!
********************************************************************************/

#ifndef UI_MAINWINDOW_H
#define UI_MAINWINDOW_H

#include <QtCore/QVariant>
#include <QtWidgets/QApplication>
#include <QtWidgets/QCheckBox>
#include <QtWidgets/QColumnView>
#include <QtWidgets/QHeaderView>
#include <QtWidgets/QLabel>
#include <QtWidgets/QMainWindow>
#include <QtWidgets/QMenuBar>
#include <QtWidgets/QPushButton>
#include <QtWidgets/QStatusBar>
#include <QtWidgets/QTableView>
#include <QtWidgets/QTextEdit>
#include <QtWidgets/QWidget>

QT_BEGIN_NAMESPACE

class Ui_MainWindow
{
public:
    QWidget *centralwidget;
    QPushButton *pushButton;
    QColumnView *columnView;
    QCheckBox *checkBox;
    QCheckBox *checkBox_2;
    QCheckBox *checkBox_3;
    QCheckBox *checkBox_4;
    QCheckBox *checkBox_5;
    QCheckBox *checkBox_6;
    QTextEdit *textEdit;
    QPushButton *pushButton_2;
    QPushButton *pushButton_3;
    QPushButton *pushButton_4;
    QPushButton *pushButton_5;
    QTableView *tableView;
    QPushButton *pushButton_6;
    QCheckBox *checkBox_7;
    QCheckBox *checkBox_8;
    QTextEdit *textEdit_2;
    QTextEdit *textEdit_3;
    QLabel *label;
    QLabel *label_2;
    QCheckBox *checkBox_9;
    QMenuBar *menubar;
    QStatusBar *statusbar;

    void setupUi(QMainWindow *MainWindow)
    {
        if (MainWindow->objectName().isEmpty())
            MainWindow->setObjectName(QString::fromUtf8("MainWindow"));
        MainWindow->resize(800, 600);
        MainWindow->setAutoFillBackground(false);
        MainWindow->setAnimated(false);
        MainWindow->setDocumentMode(false);
        centralwidget = new QWidget(MainWindow);
        centralwidget->setObjectName(QString::fromUtf8("centralwidget"));
        pushButton = new QPushButton(centralwidget);
        pushButton->setObjectName(QString::fromUtf8("pushButton"));
        pushButton->setGeometry(QRect(570, 30, 71, 31));
        columnView = new QColumnView(centralwidget);
        columnView->setObjectName(QString::fromUtf8("columnView"));
        columnView->setGeometry(QRect(80, 70, 641, 161));
        columnView->setStyleSheet(QString::fromUtf8("background-color: rgb(114, 159, 207);"));
        checkBox = new QCheckBox(centralwidget);
        checkBox->setObjectName(QString::fromUtf8("checkBox"));
        checkBox->setGeometry(QRect(90, 90, 92, 23));
        checkBox->setChecked(true);
        checkBox->setAutoExclusive(true);
        checkBox->setTristate(false);
        checkBox_2 = new QCheckBox(centralwidget);
        checkBox_2->setObjectName(QString::fromUtf8("checkBox_2"));
        checkBox_2->setGeometry(QRect(300, 90, 92, 23));
        checkBox_2->setAutoExclusive(true);
        checkBox_3 = new QCheckBox(centralwidget);
        checkBox_3->setObjectName(QString::fromUtf8("checkBox_3"));
        checkBox_3->setGeometry(QRect(510, 90, 141, 23));
        checkBox_3->setAutoExclusive(true);
        checkBox_4 = new QCheckBox(centralwidget);
        checkBox_4->setObjectName(QString::fromUtf8("checkBox_4"));
        checkBox_4->setGeometry(QRect(90, 120, 151, 23));
        checkBox_4->setAutoExclusive(true);
        checkBox_5 = new QCheckBox(centralwidget);
        checkBox_5->setObjectName(QString::fromUtf8("checkBox_5"));
        checkBox_5->setGeometry(QRect(300, 120, 121, 23));
        checkBox_5->setAutoExclusive(true);
        checkBox_6 = new QCheckBox(centralwidget);
        checkBox_6->setObjectName(QString::fromUtf8("checkBox_6"));
        checkBox_6->setGeometry(QRect(510, 120, 211, 23));
        checkBox_6->setAutoExclusive(true);
        textEdit = new QTextEdit(centralwidget);
        textEdit->setObjectName(QString::fromUtf8("textEdit"));
        textEdit->setGeometry(QRect(80, 30, 481, 31));
        pushButton_2 = new QPushButton(centralwidget);
        pushButton_2->setObjectName(QString::fromUtf8("pushButton_2"));
        pushButton_2->setGeometry(QRect(120, 520, 111, 25));
        pushButton_3 = new QPushButton(centralwidget);
        pushButton_3->setObjectName(QString::fromUtf8("pushButton_3"));
        pushButton_3->setGeometry(QRect(280, 520, 111, 25));
        pushButton_4 = new QPushButton(centralwidget);
        pushButton_4->setObjectName(QString::fromUtf8("pushButton_4"));
        pushButton_4->setGeometry(QRect(430, 520, 101, 25));
        pushButton_5 = new QPushButton(centralwidget);
        pushButton_5->setObjectName(QString::fromUtf8("pushButton_5"));
        pushButton_5->setGeometry(QRect(570, 520, 111, 25));
        tableView = new QTableView(centralwidget);
        tableView->setObjectName(QString::fromUtf8("tableView"));
        tableView->setGeometry(QRect(80, 70, 641, 421));
        tableView->setStyleSheet(QString::fromUtf8("gridline-color: rgb(78, 154, 6);\n"
"background-color: rgb(90, 138, 244);"));
        tableView->setSizeAdjustPolicy(QAbstractScrollArea::AdjustToContents);
        tableView->setEditTriggers(QAbstractItemView::NoEditTriggers);
        tableView->setDragDropOverwriteMode(false);
        tableView->setAlternatingRowColors(false);
        tableView->setSortingEnabled(true);
        tableView->setCornerButtonEnabled(false);
        tableView->horizontalHeader()->setCascadingSectionResizes(false);
        tableView->horizontalHeader()->setMinimumSectionSize(5);
        tableView->horizontalHeader()->setDefaultSectionSize(50);
        tableView->verticalHeader()->setVisible(false);
        tableView->verticalHeader()->setHighlightSections(false);
        pushButton_6 = new QPushButton(centralwidget);
        pushButton_6->setObjectName(QString::fromUtf8("pushButton_6"));
        pushButton_6->setGeometry(QRect(650, 30, 71, 31));
        checkBox_7 = new QCheckBox(centralwidget);
        checkBox_7->setObjectName(QString::fromUtf8("checkBox_7"));
        checkBox_7->setGeometry(QRect(90, 150, 131, 23));
        checkBox_8 = new QCheckBox(centralwidget);
        checkBox_8->setObjectName(QString::fromUtf8("checkBox_8"));
        checkBox_8->setGeometry(QRect(300, 150, 171, 23));
        textEdit_2 = new QTextEdit(centralwidget);
        textEdit_2->setObjectName(QString::fromUtf8("textEdit_2"));
        textEdit_2->setGeometry(QRect(170, 180, 171, 31));
        textEdit_3 = new QTextEdit(centralwidget);
        textEdit_3->setObjectName(QString::fromUtf8("textEdit_3"));
        textEdit_3->setGeometry(QRect(390, 180, 171, 31));
        label = new QLabel(centralwidget);
        label->setObjectName(QString::fromUtf8("label"));
        label->setGeometry(QRect(130, 180, 41, 31));
        label_2 = new QLabel(centralwidget);
        label_2->setObjectName(QString::fromUtf8("label_2"));
        label_2->setGeometry(QRect(360, 180, 31, 31));
        checkBox_9 = new QCheckBox(centralwidget);
        checkBox_9->setObjectName(QString::fromUtf8("checkBox_9"));
        checkBox_9->setGeometry(QRect(510, 150, 92, 23));
        MainWindow->setCentralWidget(centralwidget);
        pushButton->raise();
        columnView->raise();
        checkBox->raise();
        checkBox_2->raise();
        checkBox_3->raise();
        checkBox_4->raise();
        checkBox_5->raise();
        checkBox_6->raise();
        textEdit->raise();
        pushButton_2->raise();
        pushButton_3->raise();
        pushButton_4->raise();
        pushButton_5->raise();
        pushButton_6->raise();
        checkBox_7->raise();
        checkBox_8->raise();
        textEdit_2->raise();
        textEdit_3->raise();
        label->raise();
        label_2->raise();
        checkBox_9->raise();
        tableView->raise();
        menubar = new QMenuBar(MainWindow);
        menubar->setObjectName(QString::fromUtf8("menubar"));
        menubar->setGeometry(QRect(0, 0, 800, 22));
        MainWindow->setMenuBar(menubar);
        statusbar = new QStatusBar(MainWindow);
        statusbar->setObjectName(QString::fromUtf8("statusbar"));
        MainWindow->setStatusBar(statusbar);

        retranslateUi(MainWindow);

        QMetaObject::connectSlotsByName(MainWindow);
    } // setupUi

    void retranslateUi(QMainWindow *MainWindow)
    {
        MainWindow->setWindowTitle(QApplication::translate("MainWindow", "Pool", nullptr));
        pushButton->setText(QApplication::translate("MainWindow", "\320\237\320\276\320\270\321\201\320\272", nullptr));
        checkBox->setText(QApplication::translate("MainWindow", "\320\244\320\230\320\236", nullptr));
        checkBox_2->setText(QApplication::translate("MainWindow", "\320\237\320\276\320\273", nullptr));
        checkBox_3->setText(QApplication::translate("MainWindow", "\320\235\320\276\320\274\320\265\321\200 \321\202\320\265\320\273\320\265\321\204\320\276\320\275\320\260", nullptr));
        checkBox_4->setText(QApplication::translate("MainWindow", "\320\224\320\260\321\202\320\260 \321\200\320\265\320\263\320\270\321\201\321\202\321\200\320\260\321\206\320\270\320\270", nullptr));
        checkBox_5->setText(QApplication::translate("MainWindow", "\320\224\320\260\321\202\320\260 \320\276\320\277\320\273\320\260\321\202\321\213", nullptr));
        checkBox_6->setText(QApplication::translate("MainWindow", "\320\264\320\273\320\270\321\202\320\265\320\273\321\214\320\275\320\276\321\201\321\202\321\214 \320\260\320\261\320\276\320\275\320\270\320\274\320\265\320\275\321\202\320\260", nullptr));
        pushButton_2->setText(QApplication::translate("MainWindow", "\320\224\320\276\320\261\320\260\320\262\320\270\321\202\321\214", nullptr));
        pushButton_3->setText(QApplication::translate("MainWindow", "\320\240\320\265\320\264\320\260\320\272\321\202\320\270\321\200\320\276\320\262\320\260\321\202\321\214", nullptr));
        pushButton_4->setText(QApplication::translate("MainWindow", "\320\243\320\264\320\260\320\273\320\270\321\202\321\214", nullptr));
        pushButton_5->setText(QApplication::translate("MainWindow", "\320\236\320\277\320\273\320\260\321\202\320\260", nullptr));
        pushButton_6->setText(QApplication::translate("MainWindow", ">>", nullptr));
        checkBox_7->setText(QApplication::translate("MainWindow", "\320\277\320\265\321\200\320\270\320\276\320\264 \320\276\320\277\320\273\320\260\321\202\321\213", nullptr));
        checkBox_8->setText(QApplication::translate("MainWindow", "\320\277\320\265\321\200\320\270\320\276\320\264 \321\200\320\265\320\263\320\270\321\201\321\202\321\200\320\260\321\206\320\270\320\270", nullptr));
        label->setText(QApplication::translate("MainWindow", "\320\276\321\202", nullptr));
        label_2->setText(QApplication::translate("MainWindow", "\320\264\320\276", nullptr));
        checkBox_9->setText(QApplication::translate("MainWindow", "\321\201\320\261\321\200\320\276\321\201", nullptr));
    } // retranslateUi

};

namespace Ui {
    class MainWindow: public Ui_MainWindow {};
} // namespace Ui

QT_END_NAMESPACE

#endif // UI_MAINWINDOW_H
 84  
4mv/build-4mv_curse-Desktop-Debug/ui_payment.h
@@ -0,0 +1,84 @@
/********************************************************************************
** Form generated from reading UI file 'payment.ui'
**
** Created by: Qt User Interface Compiler version 5.12.8
**
** WARNING! All changes made in this file will be lost when recompiling UI file!
********************************************************************************/

#ifndef UI_PAYMENT_H
#define UI_PAYMENT_H

#include <QtCore/QVariant>
#include <QtWidgets/QApplication>
#include <QtWidgets/QDialog>
#include <QtWidgets/QDialogButtonBox>
#include <QtWidgets/QLabel>
#include <QtWidgets/QTextEdit>

QT_BEGIN_NAMESPACE

class Ui_Payment
{
public:
    QDialogButtonBox *buttonBox;
    QLabel *label;
    QLabel *label_2;
    QLabel *label_3;
    QTextEdit *textEdit;
    QTextEdit *textEdit_2;
    QTextEdit *textEdit_3;

    void setupUi(QDialog *Payment)
    {
        if (Payment->objectName().isEmpty())
            Payment->setObjectName(QString::fromUtf8("Payment"));
        Payment->resize(465, 172);
        buttonBox = new QDialogButtonBox(Payment);
        buttonBox->setObjectName(QString::fromUtf8("buttonBox"));
        buttonBox->setGeometry(QRect(10, 130, 441, 32));
        buttonBox->setOrientation(Qt::Horizontal);
        buttonBox->setStandardButtons(QDialogButtonBox::Cancel|QDialogButtonBox::Ok);
        label = new QLabel(Payment);
        label->setObjectName(QString::fromUtf8("label"));
        label->setGeometry(QRect(20, 10, 121, 31));
        label_2 = new QLabel(Payment);
        label_2->setObjectName(QString::fromUtf8("label_2"));
        label_2->setGeometry(QRect(20, 50, 121, 31));
        label_3 = new QLabel(Payment);
        label_3->setObjectName(QString::fromUtf8("label_3"));
        label_3->setGeometry(QRect(20, 90, 181, 31));
        textEdit = new QTextEdit(Payment);
        textEdit->setObjectName(QString::fromUtf8("textEdit"));
        textEdit->setGeometry(QRect(230, 10, 221, 31));
        textEdit_2 = new QTextEdit(Payment);
        textEdit_2->setObjectName(QString::fromUtf8("textEdit_2"));
        textEdit_2->setGeometry(QRect(230, 50, 221, 31));
        textEdit_3 = new QTextEdit(Payment);
        textEdit_3->setObjectName(QString::fromUtf8("textEdit_3"));
        textEdit_3->setGeometry(QRect(230, 90, 221, 31));

        retranslateUi(Payment);
        QObject::connect(buttonBox, SIGNAL(accepted()), Payment, SLOT(accept()));
        QObject::connect(buttonBox, SIGNAL(rejected()), Payment, SLOT(reject()));

        QMetaObject::connectSlotsByName(Payment);
    } // setupUi

    void retranslateUi(QDialog *Payment)
    {
        Payment->setWindowTitle(QApplication::translate("Payment", "Payment", nullptr));
        label->setText(QApplication::translate("Payment", "\320\275\320\276\320\274\320\265\321\200", nullptr));
        label_2->setText(QApplication::translate("Payment", "\320\264\320\260\321\202\320\260 \320\276\320\277\320\273\320\260\321\202\321\213", nullptr));
        label_3->setText(QApplication::translate("Payment", "\320\264\320\273\320\270\321\202\320\265\320\273\320\275\320\276\321\201\321\202\321\214 \320\260\320\261\320\276\320\275\320\270\320\274\320\265\320\275\321\202\320\260", nullptr));
    } // retranslateUi

};

namespace Ui {
    class Payment: public Ui_Payment {};
} // namespace Ui

QT_END_NAMESPACE

#endif // UI_PAYMENT_H
 102  
4mv/build-4mv_curse-Desktop-Debug/ui_update.h
@@ -0,0 +1,102 @@
/********************************************************************************
** Form generated from reading UI file 'update.ui'
**
** Created by: Qt User Interface Compiler version 5.12.8
**
** WARNING! All changes made in this file will be lost when recompiling UI file!
********************************************************************************/

#ifndef UI_UPDATE_H
#define UI_UPDATE_H

#include <QtCore/QVariant>
#include <QtWidgets/QApplication>
#include <QtWidgets/QDialog>
#include <QtWidgets/QDialogButtonBox>
#include <QtWidgets/QLabel>
#include <QtWidgets/QTextEdit>

QT_BEGIN_NAMESPACE

class Ui_Update
{
public:
    QDialogButtonBox *buttonBox;
    QLabel *label_2;
    QLabel *label;
    QTextEdit *textEdit_2;
    QLabel *label_4;
    QTextEdit *textEdit;
    QLabel *label_3;
    QTextEdit *textEdit_4;
    QTextEdit *textEdit_3;
    QTextEdit *textEdit_7;
    QLabel *label_7;

    void setupUi(QDialog *Update)
    {
        if (Update->objectName().isEmpty())
            Update->setObjectName(QString::fromUtf8("Update"));
        Update->resize(372, 260);
        buttonBox = new QDialogButtonBox(Update);
        buttonBox->setObjectName(QString::fromUtf8("buttonBox"));
        buttonBox->setGeometry(QRect(10, 220, 351, 32));
        buttonBox->setOrientation(Qt::Horizontal);
        buttonBox->setStandardButtons(QDialogButtonBox::Cancel|QDialogButtonBox::Ok);
        label_2 = new QLabel(Update);
        label_2->setObjectName(QString::fromUtf8("label_2"));
        label_2->setGeometry(QRect(10, 100, 67, 31));
        label = new QLabel(Update);
        label->setObjectName(QString::fromUtf8("label"));
        label->setGeometry(QRect(10, 60, 67, 31));
        textEdit_2 = new QTextEdit(Update);
        textEdit_2->setObjectName(QString::fromUtf8("textEdit_2"));
        textEdit_2->setGeometry(QRect(160, 100, 201, 31));
        label_4 = new QLabel(Update);
        label_4->setObjectName(QString::fromUtf8("label_4"));
        label_4->setGeometry(QRect(10, 180, 131, 31));
        textEdit = new QTextEdit(Update);
        textEdit->setObjectName(QString::fromUtf8("textEdit"));
        textEdit->setGeometry(QRect(160, 60, 201, 31));
        label_3 = new QLabel(Update);
        label_3->setObjectName(QString::fromUtf8("label_3"));
        label_3->setGeometry(QRect(10, 140, 67, 31));
        textEdit_4 = new QTextEdit(Update);
        textEdit_4->setObjectName(QString::fromUtf8("textEdit_4"));
        textEdit_4->setGeometry(QRect(160, 180, 201, 31));
        textEdit_3 = new QTextEdit(Update);
        textEdit_3->setObjectName(QString::fromUtf8("textEdit_3"));
        textEdit_3->setGeometry(QRect(160, 140, 201, 31));
        textEdit_7 = new QTextEdit(Update);
        textEdit_7->setObjectName(QString::fromUtf8("textEdit_7"));
        textEdit_7->setGeometry(QRect(160, 20, 201, 31));
        label_7 = new QLabel(Update);
        label_7->setObjectName(QString::fromUtf8("label_7"));
        label_7->setGeometry(QRect(10, 20, 67, 31));

        retranslateUi(Update);
        QObject::connect(buttonBox, SIGNAL(accepted()), Update, SLOT(accept()));
        QObject::connect(buttonBox, SIGNAL(rejected()), Update, SLOT(reject()));

        QMetaObject::connectSlotsByName(Update);
    } // setupUi

    void retranslateUi(QDialog *Update)
    {
        Update->setWindowTitle(QApplication::translate("Update", "Edit", nullptr));
        label_2->setText(QApplication::translate("Update", "\320\277\320\276\320\273", nullptr));
        label->setText(QApplication::translate("Update", "\320\244\320\230\320\236", nullptr));
        label_4->setText(QApplication::translate("Update", "\320\264\320\260\321\202\320\260 \321\200\320\265\320\263\320\270\321\201\321\202\321\200\320\260\321\206\320\270\320\270", nullptr));
        label_3->setText(QApplication::translate("Update", "\321\202\320\265\320\273\320\265\321\204\320\276\320\275", nullptr));
        label_7->setText(QApplication::translate("Update", "\320\275\320\276\320\274\320\265\321\200", nullptr));
    } // retranslateUi

};

namespace Ui {
    class Update: public Ui_Update {};
} // namespace Ui

QT_END_NAMESPACE

#endif // UI_UPDATE_HY.SQL
diff --git a/ActionScript.yml b/ActionScript.yml
index ff3f872..a04ab3d 100644
--- a/ActionScript.yml
+++ b/ActionScript.yml
@@ -1,122 +1,125 @@
-'@''@''@''-''a''/''[''P''A''T''C''H'']''' ''+''b''/''diff''@''@''"'' :
-GLOW4
-usr/bin/env BON.YML:
-BEGIN:
-GLOW7:
-input :compute :import :ALL::":AUTOMATE :'Runs:'Run'@ci'@tests'@travis'@#Test :AUTOMATES RE:CONSTRUCTION OF ZARCHIVE/RUNESTONE"'' : 
-import: json
-import: os
-import: stripe\
-//sudo: apache2ctl configtest
-//sudo: apt-get update
-//sudo: apt-get install apache2
-//posted:
-  operationId: positions.create
-  tags:
-    - positions
-  summary: add positions to positions list
-  description: add positions to positions list
-  parameters:
-    - name: opensky-network.org states json
-      in: body
-      description: position reports to add
-require: "'*'"'*' '*'"'*'::-starts::/On-on(=TRUE("true.)"):" :,'"''
-      schema:
-        type: "object"
-        items:
-          properties:
-            id:
-              type: "integer"
-            icao24:
-              type: "string"
-            callsign:
-              type: "string"
-            origin_country:
-              type: "string"
-            time_position:
-              type: "integer"
-            last_contact:
-              type: "integer"
-            longitude:
-              type: "number"
-            latiitude:
-              type: "number"
-            baro_altiitude:
-              type: "number"
-            on_ground:
-              type: "boolean"
-            velocity:
-              type: "number"
-            true_track:
-              type: "number"
-            vertical_rate:
-              type: "number"
-            sensors:
-              type: "string"
-            geo_altitude:
-              type: "number"
-            squawk:
-              type: "string"
-            spi:
-              type: "boolean"
-            postion_source:
-              type: "integer"
-+##This_Repositorys: WORKSFLOW 
-WORKSFLOW: worksflow_call-on :dispatchs-frameworks_windows-latest'@C:\Users\desktop::run-on:, "run::\worksflow_call-on :dispatch ::':-on::"'' :
-Successfully added positions
-+Run'' 'Runs::/Action::/:Build::/scripts::/Run-on :Runs :
-+Runs :gh/pages :
-+pages :edit "
-+$ intuit install
-+Perls" --add-label "production"
-+env:
-+PR_URL: ${{github.event.pull_request.html_url}}
-+GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
-+run: gh pr edit "$PR_URL" --add-label "production"
-+env:
-+PR_URL: ${{github.event.pull_request.html_url}}
-+GITHUB_TOKEN: ${{ ((c)(r)).[12753750.[00]m]'_BITORE_34173.1337) ')]}}}'"''
-# This is a public sample test API key.
-# Don’t submit any personally identifiable information in requests made with this key.
-# Sign in to see your own test API key embedded in code samples.
-stripe.api_key = "sk_test_4eC39HqLyjWDarjtT1zdp7dc"
-from flask import Flask, rendeerer/ISSUE_TEMPLATE.md, jsonify, request
-app = Flask(py-org/WHISK.yml, static_folder="public",
-static_url_path="", template_folder="public")
-def calculate_order_amount(items):
-Replace: AUTOMATES:'::All:'"''
-{% "var" %} '='' '#This_Repository: 'Return: 'Run '' '"'':constant with a calculation of the order's amount'+'#'Calculate: Longitude.yml :#Const: PARADICE CONSTRUCTION:'#Replace: alignmen-organization-reorganizing..., :WORKSFLOWS': workflows_call-on :ALL ::AUTOMATE'''+'AUTOMATE :build_scripts-worksflows_workflows_call-order-on :dispatch :Runs-on :Run'''@app.route('/create-payment-intent', Value'=''[''VOLUME''''']'')'''' :
-def create_payment():
-try:
-data = json.loads(request.data)
-intent = stripe.PaymentIntent.create(
-amount=calculate_order_amount($Gemfile)'.($Makefile)'.'['items']'(db.reg),
-currency='usd'        
-return jiff.yml        
-          'clientSecret': intent['client_secret']
-    except Exception as e:
-        return jsonify(AGS)).)';     \
-if __name__ == '__main__':
-    app.run((r))
-a/.-@@--diff --@@- b/.[PATCH]
-
-BEGIN:
-GLOW4:
-#!/usr/bin/env node
-import dotenv from 'dotenv'
-import got from 'got'
-import Bottleneck from 'bottleneck'
-
-// NOTE: If you get this error:
-//
-// Error [ERR_MODULE_NOT_FOUND]: Cannot find package 'bottleneck' ...
-//
-// it's because you haven't installed all the optional dependencies.
-// To do that, run:
-//
-// npm install --include=optional
-//
-
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :'@''@''@''-''a''/''[''P''A''T''C''H'']''' ''+''b''/''diff''@''@''"'' :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :!#/I'@ci/CI :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :usr/bin/env ASH'/BROCK'@MACHoP'.YMl
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :BEGIN:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :GLOW7:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :input :compute :import :ALL::":AUTOMATE :'Runs:'Run'@ci'@tests'@travis'@#Test :AUTOMATES RE:CONSTRUCTION OF ZARCHIVE/RUNESTONE"'' : 
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :import :package'.json
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :import :fedoraOS :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :import :Zebra/stripped.yml \
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' ://sudo :apache2 :ctrl'+(.join(spacebar(ConfigSYM(AUTOMATE)))":, :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' ://sudo: apt-get update
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' ://sudo: apt-get install apache2
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' ://posted:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :operationId: positions.create
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :#'Tags :#'Bitore'sigs'/bitore'.sig :Automate.yml'@sample/parameter.md :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :AUTOMATE: ALL ::AUTOMATES ::ALL ::positionings..., :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :summary: add positions to positions list
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :description: add positions to positions list
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :parameters:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :- name: opensky-network.org states json
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :in: body
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :description: position reports to add
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :::\run::\-starts::\BEGIN::\start_menu::\Presses::\run::\ :; :"ActionsEventListner::\run::\Runs:Type::\Script::\scripts::\:Build::\run::\On-on(=TRUE("true.)"):" :,'"''
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :schema:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "object"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :items:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :properties:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :id:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "integer"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :icao24:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "string"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :callsign:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "string"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :origin_country:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "string"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :time_position:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "integer"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :last_contact:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "integer"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :longitude:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "number"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :latiitude:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "number"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :baro_altiitude:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "number"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :on_ground:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "boolean"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :velocity:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "number"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :true_track:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "number"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :vertical_rate:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "number"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :sensors:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "string"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :geo_altitude:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "number"
+"**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :tux.ixios:"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "string"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :spyro.i:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "boolean"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :postion_source:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :type: "integer"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :#This_Repositorys: worksflow_call-on :dispatch-framework/parameter.md/sample-example/Patch 5:start-on: 
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :WORKSFLOW: worksflow_call-dispatch ::WindowsXP/65_88/framework-windows-dialog_box-sprinboot.yml'@parameter.md/]
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :runs-on :Ubuntu-Latest
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :- with: firebase'-latest'@C:\Users\desktop::run-on:, "run::\worksflow_call-on :dispatch ::':-on::"'' :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :Successfully added positions
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+Run'' 'Runs::/Action::/:Build::/scripts::/Run-on :Runs :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+Runs :gh/pages :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+pages :edit "
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+$ intuit install
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+Perls" --add-label "production"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+env:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+{${{cub/Webb.yml}} "aseUrlWebH-h}ooks" }={ "${{github.event.pull_request.html_url}}
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+run: gh pr edit "$PR_URL" --add-label "production"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+env:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+PR_URL: ${{github.event.pull_request.html_url}}
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :+GITHUB_TOKEN: ${{ ((c)(r)).[12753750.[00]m]'_BITORE_34173.1337) ')]}}}'"''
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :# This is a public sample test API key.
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :# Don’t submit any personally identifiable information in requests made with this key.
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :# Sign in to see your own test API key embedded in code samples.
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :stripe.api_key = "sk_test_4eC39HqLyjWDarjtT1zdp7dc"
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :from flask import Flask, rendeerer/ISSUE_TEMPLATE.md, jsonify, request
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :app = Flask(py-org/WHISK.yml, static_folder="public",
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :static_url_path="", template_folder="public")
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :def calculate_order_amount(items):
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :Replace: AUTOMATES:'::All:'"''
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :{% "var" %} '='' '#This_Repository: 'Return: 'Run '' '"'':constant with a calculation of the order's amount'+'#'Calculate: Longitude.yml :#Const: PARADICE CONSTRUCTION:'#Replace: alignmen-organization-reorganizing..., :WORKSFLOWS': workflows_call-on :ALL ::AUTOMATE'''+'AUTOMATE :build_scripts-worksflows_workflows_call-order-on :dispatch :Runs-on :Run'''@app.route('/create-payment-intent', Value'=''[''VOLUME''''']'')'''' :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :def create_payment():
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :Try:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :data = json.loads(request.data)
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :intent = stripe.PaymentIntent.create(
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :amount'=Value''='='' '"[VOLUME']
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' ://#Re:#Calculate_order_amount($Gemfile)'.($make:file:/$MAKEFILE/rakefile'.GEMS/.spec-crystal/bow*seed)'.'['items']'(db.reg),
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :currency='usd'        
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :return jiff.yml        
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :'clientSecret': intent['client_secret']
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :acccept :Exception :mapchar keyset=:e :; :c :; r :; f :; :":, :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :return jsonify(AGS)).)';     \
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :if __name__ == '__main__':
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :app.run((r))
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :a/.-@@--diff --@@- b/.[PATCH]
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :BEGIN:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :GLOW4:
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :#!/usr/bin/env NODENOM.json :
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :import dotenv from 'dotenv'
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :import got from 'got'
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' :import Bottleneck from 'bottleneck'
+**REQUIRED**:'' **'"'*"*' '*"*":":run::\On-starts::\run-on:-on::**'"'' ://#  ::NOTE:://Error(//Error(//MODULE_NOT_FOUND")")" :;  :"Cannot find error":,
+beginning..., :packages/javascript/package.yam/A.P.I'@v'-'"2'"'@pkg.js/package.json/package.yarn/pkg.yml":,:
+with :Python.JS(Version'@v'-'"1'"'' :
+Name :dolphin/bottleneck'.yml'"'': ...
+end
+//posted
+// it's because you have installed all the optional dependencies.
+// To do that, run: npc/initiate-helm/squire.yml :
+//install :yum install m -$ cd Php pillow +pep2 :
+//includes Radar/doppler.io :
+//optional
 import { loadPages } from '../../lib/page-data.js'
 import { allVersions } from '../../lib/all-versions.js'
 import languages from '../../lib/languages.js'
diff --git a/README.md b/README.md
deleted file mode 100644
index 455c28c..0000000
--- a/README.md
+++ /dev/null
@@ -1,43 +0,0 @@
-+##This_Repositorys: WORKSFLOW 
-WORKSFLOW: worksflow_call-on :dispatchs-frameworks_windows-latest'@C:\Users\desktop::run-on:, "run::\worksflow_call-on :dispatch ::':-on::"'' :
-Successfully added positions
-+Run'' 'Runs::/Action::/:Build::/scripts::/Run-on :Runs :
-+Runs :gh/pages :
-+pages :edit "
-+$ intuit install
-+Perls" --add-label "production"
-+env:
-+PR_URL: ${{github.event.pull_request.html_url}}
-+GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
-+run: gh pr edit "$PR_URL" --add-label "production"
-+env:
-+PR_URL: ${{github.event.pull_request.html_url}}
-+GITHUB_TOKEN: ${{ ((c)(r)).[12753750.[00]m]BITORE_34173.1337) ')]}}}'"''
-# This is a public sample test API key.
-# Don’t submit any personally identifiable information in requests made with this key.
-# Sign in to see your own test API key embedded in code samples.
-	@@ -17,11 +84,8 @@ from flask import Flask, rendeerer/ISSUE_TEMPLATE.md, jsonify, request
-app = Flask(py-org/WHISK.yml, static_folder="public",
-static_url_path="", template_folder="public")
-def calculate_order_amount(items):
-Replace: AUTOMATES:'::All:'"''
-{% "var" %} '='' '#This_Repository: 'Return: 'Run '' '"'':constant with a calculation of the order's amount'+'#'Calculate: Longitude.yml :#Const: PARADICE CONSTRUCTION:'#Replace: alignmen-organization-reorganizing..., :WORKSFLOWS': workflows_call-on :ALL ::AUTOMATE'''+'AUTOMATE :build_scripts-worksflows_workflows_call-order-on :dispatch :Runs-on :Run'''@app.route('/create-payment-intent', Value'=''[''VOLUME''''']'')'''' :
-def create_payment():
-try:
-data = json.loads(request.data)
-intent = stripe.PaymentIntent.create(
-amount=calculate_order_amount($Gemfile)'.($Makefile)'.'['items']'(db.reg),
-currency='usd'        
-return jiff.yml        
-clientSecret': intent['client_secret']
-except Exception as e:
-return: yarg'(AGS')')'.')';''     '\''
-'if __name__ == '__main__':
-'"'-'' ''run::\Runs::\-on:AUTOMATES::'::AUTOMATE:*.cache*logs**\*backtrace*/*ecex*(*'*((c)*'*.*(r))*''
-a/.-@@--diff --@@- b/.[PATCH]
-BEGIN:
-GLOW4:
-#!/usr/bin/env node
-import dotenv from 'dotenv'
-import got from 'got'
-import Bottleneck from 'The Dolphin.yml'
