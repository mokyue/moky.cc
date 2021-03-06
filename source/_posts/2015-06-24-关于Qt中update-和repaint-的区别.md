title: 关于Qt中update()和repaint()的区别
date: 2015-06-24 21:24:52
categories:
- Qt
tags:
- update
- repaint
---
>【转】原创作品，允许转载。转载时请务必以超链接形式标明文章原始出处、作者信息和本声明，否则将追究法律责任。
>[http://blog.csdn.net/freezgw1985/article/details/5489316](http://blog.csdn.net/freezgw1985/article/details/5489316 "http://blog.csdn.net/freezgw1985/article/details/5489316")

`void QWidget::repaint ( int x, int y, int w, int h, bool erase = TRUE ) [槽]`
通过立即调用paintEvent()来直接重新绘制窗口部件，如果erase为真，Qt在paintEvent()调用之前擦除区域(x,y,w,h)。 
如果w是负数，它被width()-x替换，并且如果h是负数，它被height()-y替换。 如果你需要立即重新绘制，建议使用repaint()，比如在动画期间。在绝大多数情况下，update()更好，因为它允许Qt来优化速度并且防止闪烁。 

警告：如果你在一个函数中调用repaint()，而它自己又被paintEvent()调用，你也许会看到无线循环。
update()函数从来不会产生循环。
<br>
`void QWidget::update () [槽]`
更新窗口部件，当Qt回到主事件中时，它规划了所要处理的绘制事件。这样允许Qt进行优化从而得到比调用repaint()更快的速度和更少的闪烁。几次调用update()的结果通常仅仅是一次paintEvent()调用。 Qt通常在paintEvent()调用之前擦除这个窗口部件的区域，仅仅只有在WRepaintNoErase窗口部件标记被设置的时候才不会。
在这区别中关键点是：repaint()是立即调用paintEvent()，而update()是几次执行才调用一次paintEvent()。
这样update()会造成这样的结果：paintEvent()中的任务没有执行完，就又被update().paintEvent()中被积压的任务越来越多。
<br>
程序例子：
(1)问题出现时候的情况(10毫秒每次，用update()。paintEvent()积累了很多处理任务)：
``` cpp
#include<QPainter>
#include<QDebug>
#include<QMessageBox>
#include "mainwindow.h"
#include "ui_mainwindow.h"
 
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    this->showMaximized();
    i = 0;
    realWidth = this->width();
    realHeight = this->height();
    pixmap = QPixmap(realWidth,realHeight);
    connect(this,SIGNAL(haveData(QPoint)),this,SLOT(getPointAndDraw(QPoint)));
    connect(&timer,SIGNAL(timeout()),this,SLOT(getPoint()));
    timer.start(10);
}
 
MainWindow::~MainWindow()
{
    delete ui;
}
 
void MainWindow::getPoint()
{
    if(i < realWidth)
    {
        point = QPoint(i,(uint(qrand())) % realHeight);
        i++;
    }
    else
    {
        i = i % realWidth;
        point = QPoint(i,(uint(qrand())) % realHeight);
        i++;
    }
    emit haveData(point);
}
 
void MainWindow::getPointAndDraw(QPoint point)
{
    index = point.x();
    QPainter painter(&pixmap);
    painter.setPen(Qt::green);
    painter.drawLine(lastPoint,point);
    painter.setPen(Qt::black);
    painter.setBrush(Qt::red);
    painter.drawRect(index+1,0,5,realHeight);
    if(point.x() < realWidth-1)
        lastPoint = point;
    else
        lastPoint = QPoint(0,0);
  update();
  // this->repaint(index-1,0,5,realHeight);
}
 
void MainWindow::paintEvent(QPaintEvent *e)
{
    //return ;
    QPainter painter(this);
    QRect target1(0, 0, realWidth, realHeight/5);
    QRect target2(0, realHeight/5, realWidth, realHeight/5);
    QRect target3(0, 2*realHeight/5, realWidth, realHeight/5);
    QRect target4(0, 3*realHeight/5, realWidth, realHeight/5);
    QRect target5(0, 4*realHeight/5, realWidth, realHeight/5);
    QRect source(0, 0, realWidth, realHeight);
    painter.drawPixmap(target1,pixmap,source);
    painter.drawPixmap(target2,pixmap,source);
    painter.drawPixmap(target3,pixmap,source);
    painter.drawPixmap(target4,pixmap,source);
    painter.drawPixmap(target5,pixmap,source);
}
 
void MainWindow::resizeEvent(QResizeEvent *e)
{
    realWidth = this->width();
    realHeight = this->height();
}
 
void MainWindow::changeEvent(QEvent *e)
{
    QMainWindow::changeEvent(e);
    switch (e->type()) {
    case QEvent::LanguageChange:
        ui->retranslateUi(this);
        break;
    default:
        break;
    }
}
```
<br>
(2)每隔1000毫秒刷新一次，用update().一秒种有足够的时间处理paintEvent()，无积累。
``` cpp
#include<QPainter>
#include<QDebug>
#include<QMessageBox>
#include "mainwindow.h"
#include "ui_mainwindow.h"
 
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    this->showMaximized();
    i = 0;
    realWidth = this->width();
    realHeight = this->height();
    pixmap = QPixmap(realWidth,realHeight);
    connect(this,SIGNAL(haveData(QPoint)),this,SLOT(getPointAndDraw(QPoint)));
    connect(&timer,SIGNAL(timeout()),this,SLOT(getPoint()));
    timer.start(1000);
}
 
MainWindow::~MainWindow()
{
    delete ui;
}
 
void MainWindow::getPoint()
{
    if(i < realWidth)
    {
        point = QPoint(i,(uint(qrand())) % realHeight);
        i++;
    }
    else
    {
        i = i % realWidth;
        point = QPoint(i,(uint(qrand())) % realHeight);
        i++;
    }
    emit haveData(point);
}
 
void MainWindow::getPointAndDraw(QPoint point)
{
    index = point.x();
    QPainter painter(&pixmap);
    painter.setPen(Qt::green);
    painter.drawLine(lastPoint,point);
    painter.setPen(Qt::black);
    painter.setBrush(Qt::red);
    painter.drawRect(index+1,0,5,realHeight);
    if(point.x() < realWidth-1)
        lastPoint = point;
    else
        lastPoint = QPoint(0,0);
 update();
  // this->repaint(index-1,0,5,realHeight);
}
 
void MainWindow::paintEvent(QPaintEvent *e)
{
    //return ;
    QPainter painter(this);
    QRect target1(0, 0, realWidth, realHeight/5);
    QRect target2(0, realHeight/5, realWidth, realHeight/5);
    QRect target3(0, 2*realHeight/5, realWidth, realHeight/5);
    QRect target4(0, 3*realHeight/5, realWidth, realHeight/5);
    QRect target5(0, 4*realHeight/5, realWidth, realHeight/5);
    QRect source(0, 0, realWidth, realHeight);
    painter.drawPixmap(target1,pixmap,source);
    painter.drawPixmap(target2,pixmap,source);
    painter.drawPixmap(target3,pixmap,source);
    painter.drawPixmap(target4,pixmap,source);
    painter.drawPixmap(target5,pixmap,source);
}
 
void MainWindow::resizeEvent(QResizeEvent *e)
{
    realWidth = this->width();
    realHeight = this->height();
}
 
void MainWindow::changeEvent(QEvent *e)
{
    QMainWindow::changeEvent(e);
    switch (e->type()) {
    case QEvent::LanguageChange:
        ui->retranslateUi(this);
        break;
    default:
        break;
    }
}
```
<br>
(3)继续改进(10毫秒每次，用repaint()。一次repaint(),一次paintEvent(),无积累).
``` cpp
#include<QPainter>
#include<QDebug>
#include<QMessageBox>
#include "mainwindow.h"
#include "ui_mainwindow.h"
 
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    this->showMaximized();
    i = 0;
    realWidth = this->width();
    realHeight = this->height();
    pixmap = QPixmap(realWidth,realHeight);
    connect(this,SIGNAL(haveData(QPoint)),this,SLOT(getPointAndDraw(QPoint)));
    connect(&timer,SIGNAL(timeout()),this,SLOT(getPoint()));
    timer.start(10);
}
 
MainWindow::~MainWindow()
{
    delete ui;
}
 
void MainWindow::getPoint()
{
    if(i < realWidth)
    {
        point = QPoint(i,(uint(qrand())) % realHeight);
        i++;
    }
    else
    {
        i = i % realWidth;
        point = QPoint(i,(uint(qrand())) % realHeight);
        i++;
    }
    emit haveData(point);
}
 
void MainWindow::getPointAndDraw(QPoint point)
{
    index = point.x();
    QPainter painter(&pixmap);
    painter.setPen(Qt::green);
    painter.drawLine(lastPoint,point);
    painter.setPen(Qt::black);
    painter.setBrush(Qt::red);
    painter.drawRect(index+1,0,5,realHeight);
    if(point.x() < realWidth-1)
        lastPoint = point;
    else
        lastPoint = QPoint(0,0);
   this->repaint(index-1,0,5,realHeight);
}
 
void MainWindow::paintEvent(QPaintEvent *e)
{
    //return ;
    QPainter painter(this);
    QRect target1(0, 0, realWidth, realHeight/5);
    QRect target2(0, realHeight/5, realWidth, realHeight/5);
    QRect target3(0, 2*realHeight/5, realWidth, realHeight/5);
    QRect target4(0, 3*realHeight/5, realWidth, realHeight/5);
    QRect target5(0, 4*realHeight/5, realWidth, realHeight/5);
    QRect source(0, 0, realWidth, realHeight);
    painter.drawPixmap(target1,pixmap,source);
    painter.drawPixmap(target2,pixmap,source);
    painter.drawPixmap(target3,pixmap,source);
    painter.drawPixmap(target4,pixmap,source);
    painter.drawPixmap(target5,pixmap,source);
}
 
void MainWindow::resizeEvent(QResizeEvent *e)
{
    realWidth = this->width();
    realHeight = this->height();
}
 
void MainWindow::changeEvent(QEvent *e)
{
    QMainWindow::changeEvent(e);
    switch (e->type()) {
    case QEvent::LanguageChange:
        ui->retranslateUi(this);
        break;
    default:
        break;
    }
}
```
