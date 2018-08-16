# jimi
#-------------------------------------------------
#
# Project created by QtCreator 2018-08-09T16:21:31
#
#-------------------------------------------------
#简陋的图片查看器
//mypictur.pro
QT       += core gui

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

TARGET = mypictur
TEMPLATE = app

# The following define makes your compiler emit warnings if you use
# any feature of Qt which as been marked as deprecated (the exact warnings
# depend on your compiler). Please consult the documentation of the
# deprecated API in order to know how to port your code away from it.
DEFINES += QT_DEPRECATED_WARNINGS

# You can also make your code fail to compile if you use deprecated APIs.
# In order to do so, uncomment the following line.
# You can also select to disable deprecated APIs only up to a certain version of Qt.
#DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    # disables all the APIs deprecated before Qt 6.0.0


SOURCES += main.cpp\
        widget.cpp

HEADERS  += widget.h

FORMS    += widget.ui
//widget.h
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include<QLabel>
#include<QPushButton>
#include<QDebug>
#include<QMouseEvent>
#include<QWheelEvent>
#include<QKeyEvent>

#include<QScrollArea>
#include<QImage>
#include<QFileInfoList>
namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:

    Widget(QWidget *parent = 0);
    ~Widget();
public:
    QImage image;
    QString filename;
    QPixmap pix;
    int index;
    int imageAngle;
    QString path;
    QFileInfoList imgInfoList;

public:
    void imageShow();
    void getImInfoList();
public slots:
    void openpfile();
    void lastshow();
    void nextshow();
    void W_max();
    void W_min();
    void toleft();
    void toright();

private:
    Ui::Widget *ui;
    QLabel *imageLabel;
    QPushButton *btn1;
    QPushButton *btn2;
    QPushButton *btn3;
    QPushButton *btn4;
    QPushButton *btn5;
    QPushButton *btn6;
    QPushButton *btn7;
    QPoint offset;
protected:
    void mouseMoveEvent(QMouseEvent *event);
    void mouseReleaseEvent(QMouseEvent *event);
    void mouseDoubleClickEvent(QMouseEvent * event);
};


#endif // WIDGET_H
//main.cpp
#include "widget.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Widget w;
    w.show();

    return a.exec();
}
//widget.cpp
#include "widget.h"
#include "ui_widget.h"

#include<QVBoxLayout>
#include<QHBoxLayout>
#include<QString>
#include<QFileDialog>
#include<QMouseEvent>
#include<QLabel>
#include<QPixmap>
#include<QPainter>
#include<QScrollArea>
#include<QImage>
#include<QMessageBox>
#include<QString>
#include<QFileInfoList>
Widget::Widget(QWidget *parent)
    : QWidget(parent),
  ui(new Ui::Widget)
{
    ui->setupUi(this);

    setFixedSize(640,560);
    imageLabel = new QLabel(this);
    btn4 = new QPushButton("放大",this);
    btn1 = new QPushButton("上一张",this);
    btn2 =new QPushButton("打开图片",this);
    btn3 = new QPushButton("下一张",this);
    btn5 = new QPushButton("缩小",this);
    btn6 = new QPushButton("左转",this);
    btn7 = new QPushButton("右转",this);
    //设置按钮的图标和名字
    QHBoxLayout *hbox = new QHBoxLayout;
    hbox->addWidget(btn1);
    hbox->addWidget(btn2);
    hbox->addWidget(btn3);
    hbox->addWidget(btn4);
    hbox->addWidget(btn5);
    hbox->addWidget(btn6);
    hbox->addWidget(btn7);
    QVBoxLayout *vbox = new QVBoxLayout;
    vbox->addWidget(imageLabel);
    vbox->addLayout(hbox);
    setLayout(vbox);
    connect(btn2,SIGNAL(clicked()),this,SLOT(openpfile()));
    connect(btn1,SIGNAL(clicked()),this,SLOT(lastshow()));
    connect(btn3,SIGNAL(clicked()),this,SLOT(nextshow()));
    connect(btn4,SIGNAL(clicked()),this,SLOT(W_max()));
    connect(btn5,SIGNAL(clicked()),this,SLOT(W_min()));
    connect(btn6,SIGNAL(clicked()),this,SLOT(toleft()));
    connect(btn7,SIGNAL(clicked()),this,SLOT(toright()));
    QCursor cursor;//创建光标对象
    cursor.setShape(Qt::OpenHandCursor);//设置光标形状
    setCursor(cursor);//使用光标
    setMouseTracking(true);//设置光标跟踪
}

void Widget::openpfile()
{
         filename = QFileDialog::getOpenFileName(this, tr("Select image:"),".", tr("Images (*.png *.bmp *.jpg *.gif)"));
        if (filename.isEmpty())
            return;
        if (!image.load(filename)) {
            QMessageBox::information(this, tr("Error"), tr("Open file error"));
            return;
        }
        getImInfoList();
        imageShow();

}
void Widget::imageShow()
{

    QPixmap pixmap = QPixmap::fromImage(image);
    pix = pixmap;//全局pix
    QSize imagesize = pixmap.size();
    imageLabel->resize(imagesize);//重新调整label大小以适应图片大小
    imageLabel->setPixmap(pixmap);
}
void Widget::getImInfoList() {
    imgInfoList.clear();
    path = QFileInfo(filename).absolutePath();//绝对路径
    QDir dir = QFileInfo(filename).absoluteDir();
    QFileInfoList infoList = dir.entryInfoList(QDir::Files);
    QFileInfo info;
    for (int i = 0; i < infoList.count(); i++) {
        info = infoList.at(i);
        QString suffix = info.suffix();
        if (suffix == "jpg" || suffix == "bmp" || suffix == "png") {
            imgInfoList.append(info);
        }
    }

    QFileInfo curImageInfo = QFileInfo(filename);
    for (int j = 0; j < imgInfoList.count(); j++) {

        info = imgInfoList.at(j);
        if (info.fileName() == curImageInfo.fileName()) {
            index = j;
    }
    }
}


void Widget::nextshow()//下一张
{
    index = index + 1;
        int count = imgInfoList.count();
        if (index == count) {
            index = 0;
     }
        filename.clear();
        filename.append(path);
        filename += "/";
        filename += imgInfoList.at(index).fileName();
        if (!image.load(filename)) {
            QMessageBox::information(this,tr("Error"),tr("Open file Error"));
            return;
        }
        imageShow();

}
void Widget::toleft()//向左转
{
    QMatrix matrix;
        QPixmap pixmap;
        imageAngle += 1;
        imageAngle = imageAngle % 4;
        matrix.rotate(imageAngle*90);
        image.load(filename);
        image = image.transformed(matrix);
        imageShow();
}
void Widget::toright()//向右转
{
    QMatrix matrix;
        QPixmap pixmap;
        imageAngle += 3;
        imageAngle = imageAngle % 4;
        matrix.rotate(imageAngle * 90);
        image.load(filename);
        image = image.transformed(matrix);
        imageShow();
}

void Widget::lastshow()//上一张
{
    index = index - 1;
        int count = imgInfoList.count();
        if (index == -1)
            index = count - 1;
        filename.clear();
        filename.append(path);
        filename += "/";
        filename += imgInfoList.at(index).fileName();

        if (!image.load(filename)) {
            QMessageBox::information(this,tr("Error"),tr("Open file error"));
            return;
        }
        imageShow();

}

void Widget::W_max()//放大
{
    QMatrix matrix;
        QPixmap pixmap;
        image.load(filename);
        matrix.rotate(imageAngle * 90);
        image = image.transformed(matrix);
        pixmap = QPixmap::fromImage(image);
        pixmap = pixmap.scaled(pix.width()*1.2, pix.height()*1.2, Qt::KeepAspectRatio);
        pix = pix.scaled(pix.width()*1.2, pix.height()*1.2, Qt::KeepAspectRatio);

        imageLabel->setPixmap(pixmap);
}
void Widget::W_min()//缩小
{
    QMatrix matrix;
         QPixmap pixmap;
         image.load(filename);
         matrix.rotate(imageAngle * 90);
         image = image.transformed(matrix);
         pixmap = QPixmap::fromImage(image);
         pixmap = pixmap.scaled(pix.width()*0.8, pix.height()*0.8, Qt::KeepAspectRatio);
         pix = pix.scaled(pix.width()*0.8, pix.height()*0.8, Qt::KeepAspectRatio);

        imageLabel->setPixmap(pixmap);
}
void Widget::mouseDoubleClickEvent(QMouseEvent *event)//鼠标双击事件
{
    if(event->button()==Qt::LeftButton){
        if(windowState()!=Qt::WindowFullScreen)
            setWindowState(Qt::WindowFullScreen);
        else setWindowState(Qt::WindowNoState);
    }
}

void Widget::mouseMoveEvent(QMouseEvent *event)
{
    if(event->buttons() & Qt::LeftButton){
        QPoint temp;
        temp=event->globalPos() - offset;
        move(temp);
    }
}
void Widget::mouseReleaseEvent(QMouseEvent *event)
{
    Q_UNUSED(event);
    QApplication::restoreOverrideCursor();
}

Widget::~Widget(){
   delete ui;
}
