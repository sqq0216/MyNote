# PyQt4简介

## 安装(前提：已经安装了python2.7)

### 1.安装SIP

- 下载sip-4.19.17［网址］（https://www.riverbankcomputing.com/news）
- sudo tar -zxvf sip...tar.gz
- cd sip-4.19.17/
- python configure.py
- make
- make install
- 如果出错可执行：sudo apt-get install python2.7-dev

### 2.安装PyQt4

- 下载依赖包：sudo apt-get install qt4-dev-tools qt4-doc qt4-qtconfig qt4-demos qt4-designer libqwt5-qt4 libqwt5-qt4-dev
- 下载PyQt4_gpl_x11-4.12.3［网址］（https://www.riverbankcomputing.com/news）
- sudo tar -zxvf PyQt4_gpl_x11-4.12.3．tar.gz
- cd PyQt4_gpl_x11-4.12.3/
- python configure.py
- make
- make install
- 注：sip和pyqt需要匹配

## 使用

## Q&A
