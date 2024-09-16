# Лабораторная работа №1

# Создание веб-приложений. Сервлеты

## Цель работы

В лабораторной работе №6 необходимо создать веб-приложение (Java 2 Enterprise Edition), разместив его на веб-сервере Tomcat.

## Указания к работе

На настоящий момент более 90% корпоративных систем поддерживают Java 2 Enterprise Edition. Эта платформа позволяет быстро и без особых издержек объединить возможности сети Интернет и корпоративных информационных систем в сетевых приложениях на основе [клиент-серверной архитетектуры](http://www.4stud.info/networking/lecture5.html).

**Сервлеты** – это компоненты приложений Java 2 Platform Enterprise Edition (J2EE), выполняющиеся на стороне сервера (см. [сервер приложений](http://www.4stud.info/networking/application-server.html)), способные обрабатывать клиентские запросы и динамически генерировать ответы на них. Наибольшее распространение получили сервлеты, обрабатывающие клиентские запросы по [протоколу HTTP](http://www.4stud.info/web-programming/protocol-http.html). Сервлет может применяться, например, для создания серверного приложения, получающего от клиента запрос, анализирующего его и делающего выборку данных из базы данных, а также пересылающего клиенту [страницу HTML](http://www.4stud.info/web-programming/work1.html), сгенерированную с помощью JSP на основе полученных данных.

Все сервлеты реализуют общий интерфейс Servlet. Для обработки HTTP-запросов можно воспользоваться в качестве базового класса абстрактным классом HttpServlet. Базовая часть классов JSDK помещена в пакет javax.servlet. Однако класс HttpServlet и все, что с ним связано, располагаются на один уровень ниже в пакете javax.servlet.http.

Жизненный цикл сервлета начинается с его загрузки в память контейнером сервлетов при старте либо в ответ на первый запрос. Далее происходят инициализация, обслуживание запросов и завершение существования.

Первым вызывается метод init(). Он дает сервлету возможность инициализировать данные и подготовиться для обработки запросов. Чаще всего в этом методе программисты помещают код, кэширующий данные фазы инициализации.

После этого сервлет можно считать запущенным, он находится в ожидании запросов от клиентов. Появившийся запрос обслуживается методом service() сервлета, а все параметры запроса упаковываются в объект ServletRequest, который передается в качестве первого параметра методу service(). Второй параметр метода – объект ServletResponse. В этот объект упаковываются выходные данные в процессе формирования ответа клиенту. Каждый новый запрос приводит к новому вызову метода service(). В соответствии со спецификацией JSDK, метод service() должен уметь обрабатывать сразу несколько запросов, т.е. быть синхронизирован для выполнения в многопоточных средах. Если же нужно избежать множественных запросов, сервлет должен реализовать интерфейс SingleThreadModel, который не содержит ни одного метода и только указывает серверу об однопоточной природе сервлета. При обращении к такому сервлету каждый новый запрос будет ожидать в очереди, пока не завершится обработка предыдущего запроса.

После завершения выполнения сервлета контейнер сервлетов вызывает метод destroy(), в теле которого следует помещать код освобождения занятых сервлетом ресурсов.

Интерфейсом Servlet предусмотрена реализация еще двух методов: getServletConfig() и getServletInfo(). Первый возвращает объект типа ServletConfig, содержащий параметры конфигурации сервлета, а второй – строку, описывающую назначение сервлета.

При разработке сервлетов в качестве базового класса в большинстве случаев используют не интерфейс Servlet, а класс HttpServlet, отвечающий за обработку запросов HTTP.

Класс HttpServlet имеет реализованный метод service(), служащий диспетчером для других методов, каждый из которых обрабатывает методы доступа к ресурсам. В спецификации HTTP определены следующие методы: GET, HEAD, POST, PUT, DELETE, OPTIONS и TRACE. Наиболее часто употребляются методы GET и POST, с помощью которых на сервер передаются запросы, а также параметры для их выполнения.

При использовании метода GET (по умолчанию) параметры передаются как часть URL, значения могут выбираться из полей формы или передаваться непосредственно через URL. При этом запросы кэшируются и имеют ограничения на размер. При использовании метода POST (method=POST) параметры (поля формы) передаются в содержимом HTTP-запроса и упакованы согласно полю заголовка Content-Type. По умолчанию в формате: <имя>=<значение>&<имя>=<значение>&...

Однако форматы упаковки параметров могут быть самые разные, например: в случае передачи файлов с использованием формы `enctype="multipart/form-data".`

В задачу метода service() класса HttpServlet входит анализ полученного через запрос метода доступа к ресурсам и вызов метода, имя которого сходно с названием метода доступа к ресурсам, но перед именем добавляется префикс do: doGet() или doPost(). Кроме этих методов могут использоваться методы: doHead(), doPut(), doDelete(), doOptions() и doTrace(). Разработчик должен переопределить нужный метод, разместив в нем функциональную логику.

В следующем примере приведен готовый к выполнению «шаблон» сервлета:

```
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
public class MyServlet extends HttpServlet {
/** Constructor of the object. */
public MyServlet() {
super();
}
/** Destruction of the servlet. */
public void destroy() {
super.destroy(); // Just puts "destroy" string in log
// Put your code here
}
/** The doDelete method of the servlet.
* This method is called when a HTTP delete request is received.
* @param request the request send by the client to the erver
* @param response the response send by the server to the client
* @throws ServletException if an error occurred
* @throws IOException if an error occurred */
public void doDelete(HttpServletRequest request,
HttpServletResponse response)
throws ServletException, IOException {
// Put your code here
}
/** The doGet method of the servlet.
* This method is called when a form has its tag value method equals to get.
* @param request the request send by the client to the server
* @param response the response send by the server to the client
* @throws ServletException or @throws IOException if an error occurred */
public void doGet(HttpServletRequest request, HttpServletRe-sponse response)
throws ServletException, IOException {
response.setContentType("text/html");
this.preventCaching(request, response);
PrintWriter out = response.getWriter();
out.println( "<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\">");
out.println("<HTML><HEAD>");
out.println("<TITLE>A Servlet in GET</TITLE>");
out.println("</HEAD><BODY>");
out.print(" This is <B>");
out.print(this.getClass().getName());
out.println("</B>, using the <B>GET</B> method<BR>");
out.println("</BODY></HTML>");
out.flush();
out.close();
}
/** The doPost method of the servlet.
* This method is called when a form has its tag value method equals to post.
* @param request the request send by the client to the server
* @param response the response send by the server to the client
* @throws ServletException or @throws IOException if an error occurred */
public void doPost(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException {
response.setContentType("text/html");
this.preventCaching(request, response);
PrintWriter out = response.getWriter();
out.println("<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\">");
out.println("<HTML><HEAD>");
out.println("<TITLE>A Servlet in POST</TITLE>");
out.println("</HEAD><BODY>");
out.print("This is <B>");
out.print(this.getClass().getName());
out.println("</B>, using the <B>POST</B> method");
out.println("</BODY></HTML>");
out.flush();
out.close();
}
/** The doPut method of the servlet.
* This method is called when a HTTP put request is received.
* @param request the request send by the client to the server
* @param response the response send by the server to the client
* @throws ServletException or @throws IOException if an error occurred */
public void doPut(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException {
// Put your code here
}
/** Returns information about the servlet, such as
* author, version, and copyright.
* @return String information about this servlet */
public String getServletInfo() {
return "This is my default servlet created by Eclipse";
}
/* Initilaisation of the servlet.
* @throws ServletException if an error occure */
public void init() throws ServletException {
// Put your code here
}
/** Prevents navigator from caching data.
* @param request the request send by the client to the server
* @param response the response send by the server to the client
*/
protected void preventCaching(HttpServletRequest request,HttpServletResponse response) {
/* see http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html */
String protocol = request.getProtocol();
if ("HTTP/1.0".equalsIgnoreCase(protocol)) {
response.setHeader("Pragma", "no-cache");
} else if ("HTTP/1.1".equalsIgnoreCase(protocol)) {
response.setHeader("Cache-Control", "no-cache");
}
response.setDateHeader("Expires", 0);
}
}
```

## Задания к лабораторной работе

Создать сервлет и взаимодействующие с ним пакеты Java-классов и HTML-документов. Готовое веб-приложение разместить на сервере Tomcat (см. "[Установка и настройка TomCat](http://www.4stud.info/networking/work10.html)").

1. Генерация таблиц по переданным параметрам: заголовок, количество строк и столбцов, цвет фона.
2. Вычисление тригонометрических функций в градусах и радианах с указанной точностью. Выбор функций должен осуществляться через выпадающий список.
3. Поиск слова, введенного пользователем. Поиск и определение частоты встречаемости осуществляется в текстовом файле, расположенном на сервере.
4. Вычисление объемов тел (параллелепипед, куб, сфера, тетраэдр, тор, шар, эллипсоид и т.д.) с точностью и параметрами, указываемыми пользователем.
5. Поиск и (или) замена информации в коллекции по ключу (значению).
6. Выбор текстового файла из архива файлов по разделам (поэзия, проза, фантастика и т.д.) и его отображение.
7. Выбор изображения по тематике (природа, автомобили, дети и т.д.) и его отображение.
8. Информация о среднесуточной температуре воздуха за месяц задана в виде списка, хранящегося в файле. Определить: а) среднемесячную температуру воздуха; б) количество дней, когда температура была выше среднемесячной; в) количество дней, когда температура опускалась ниже ; г) три самых теплых дня.
9. Реализация адаптивного теста из цепочки в 3 – 4 вопроса.
10. Вывод фрагментов текстов шрифтами различного размера. Размер шрифта и количество строк задается на стороне клиента.
11. Информация о точках на плоскости хранится в файле. Выбрать все точки, наиболее приближенные к заданной прямой. Параметры прямой и максимальное расстояние от точки до прямой вводятся на стороне клиента.
12. Осуществить сортировку введенного пользователем массива целых чисел. Числа вводятся через запятую.
13. Осуществить форматирование выбранного пользователем текстового файла, так чтобы все абзацы имели отступ ровно 3 пробела, а длина каждой строки была ровно 80 символов и не имела начальными и конечными символами пробел.
