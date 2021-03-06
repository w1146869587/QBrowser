# QBrowser

## Contents

1. [Before we start](#before-we-start)
2. [IDEs and plugins used](#ides-and-plugins-used)
3. [Custom elements](#custom-elements)
4. [References](#references)

## Before we start

![Main window](https://i.imgur.com/meXOuL4.png)
*Main window*

The application is developed as the part of the project for programming course at [Irkutsk National Research Techincal University](http://www.istu.edu/eng/).

## IDEs and plugins used
- Qt Creator Community Edition ([Download](https://www.qt.io/download))
- QtWebkit ([Download](https://github.com/wkhtmltopdf/qtwebkit))
- OpenSSL ([Download](https://www.openssl.org/source/))

## Custom elements

There are a few custom elements I created to follow the task:
1. [Custom address bar displaying loading progress](#custom-address-bar)
2. [Source code of a web page](#source-code-of-a-web-page)

### Custom address bar

Related .cpp files:
- [urllineedit.h](https://github.com/xtenzQ/QBrowser/blob/master/urllineedit.h)
- [urllineedit.cpp](https://github.com/xtenzQ/QBrowser/blob/master/urllineedit.cpp)

To follow the basic requirement of the task to have at least one custom element I decided to create my own address bar displaying loading progress as main feature. It's very simple and based on default QLineEdit component.

![Custom address bar](https://i.imgur.com/kJbOGDw.png)

First of all, we have to repaint the original QLineEdit before drawing the progress bar:
```C++
QLineEdit::paintEvent(e);
```
Then we draw a rectangle in the background:
```C++
QPainter painter(this);
QStyleOptionFrame panel;
initStyleOption(&panel);
QRect backgroundRect = style()->subElementRect(QStyle::SE_LineEditContents, &panel, this);
```
If webView is set we're getting the load progress then drawing a rectangle, filling it with a color and displaying loading progress over the bar:
```C++
if (m_webView) {
        // Get loading progress
        int progress = value;        
        QColor loadingColor = QColor(46, 204, 113);
        painter.setBrush(generateGradient(loadingColor));
        // Transparent borders
        painter.setPen(Qt::transparent);
        // Get width of the bar depending on the load
        int mid = (backgroundRect.width() + 50) / 100.0f * progress;
        QRect progressRect(backgroundRect.x() - 25, backgroundRect.y() - 5, mid, backgroundRect.height() + 10);
        // Draw a rectangle
        painter.drawRect(progressRect);
        // Repaint text over the bar
        painter.setPen(Qt::black);
        QRect newQR(backgroundRect.x() + 2, backgroundRect.y() + 1, mid, backgroundRect.height());
        painter.drawText(newQR, url().toString());
    }
```
All together:
```C++
void UrlLineEdit::paintEvent(QPaintEvent *e) {
    QLineEdit::paintEvent(e);
    QPainter painter(this);
    QStyleOptionFrame panel;
    initStyleOption(&panel);
    QRect backgroundRect = style()->subElementRect(QStyle::SE_LineEditContents, &panel, this);
    if (m_webView) {
        int progress = value;
        QColor loadingColor = QColor(46, 204, 113);
        painter.setBrush(generateGradient(loadingColor));
        painter.setPen(Qt::transparent);
        int mid = (backgroundRect.width() + 50) / 100.0f * progress;
        QRect progressRect(backgroundRect.x() - 25, backgroundRect.y() - 5, mid, backgroundRect.height() + 10);
        painter.drawRect(progressRect);
        painter.setPen(Qt::black);
        QRect newQR(backgroundRect.x() + 2, backgroundRect.y() + 1, mid, backgroundRect.height());
        painter.drawText(newQR, url().toString());
    }
}
```
There's also an animated icon to display the process of loading:
```C++
UrlLineEdit::UrlLineEdit(QWidget *parent)
    : QLineEdit(parent)
{
    ////
    /// BLOCK OF CODE
    ///
    movie = new QMovie(":load.gif");
    connect(movie,SIGNAL(frameChanged(int)),this,SLOT(setLoadingIcon(int)));
    // infinity loop
    if (movie->loopCount() != -1)
        connect(movie,SIGNAL(finished()),movie,SLOT(start()));
    movie->start();
}
```

### Source code of a web page

Related .cpp files:
- [htmlhighlighter.h](https://github.com/xtenzQ/QBrowser/blob/master/htmlhighlighter.h)
- [htmlhighlighter.cpp](https://github.com/xtenzQ/QBrowser/blob/master/htmlhighlighter.cpp)

For a second custom element I decided to create a component showing the source code of the page with syntax highlighter (based on Eugene Legotskoy code [[1](#references)]). 

![Source code](https://imgur.com/PzgyIRF.png)

There's a part of MainWindow class responsible for window calling:

```C++
void MyMainWindow::viewSource() {
    QNetworkAccessManager* accessManager = webView->page()->networkAccessManager();
    QNetworkRequest request(webView->url());
    QNetworkReply* reply = accessManager->get(request);
    connect(reply, SIGNAL(finished()), this, SLOT(slotSourceDownloaded()));
}

void MyMainWindow::slotSourceDownloaded() {
    QNetworkReply* reply = qobject_cast<QNetworkReply*>(const_cast<QObject*>(sender()));

    QDialog* myDialog = new QDialog();

    myDialog->setWindowFlags(Qt::Window);
    myDialog->resize(700, 500);

    QTextEdit* textEdit = new QTextEdit(NULL);
    myDialog->setWindowTitle(tr("Source code of ") + (webView->url()).toString());
    myDialog->setAttribute(Qt::WA_DeleteOnClose);
    myDialog->setWindowIcon(QIcon(QStringLiteral(":sourceCode.png")));

    QGridLayout *dialogLayout = new QGridLayout();
    dialogLayout->addWidget(textEdit);

    myDialog->setLayout(dialogLayout);

    textEdit->setAttribute(Qt::WA_DeleteOnClose);
    textEdit->setPlainText(reply->readAll());

    m_htmlHightLighter = new HtmlHighLighter(textEdit->document());
    textEdit->setReadOnly(true);

    myDialog->show();
    reply->deleteLater();
}
```
# References
1. [Qt/C++ - Lesson 058. Syntax highlighting of HTML code in QTextEdit](https://evileg.com/en/post/218/) by Eugene Legotskoy
