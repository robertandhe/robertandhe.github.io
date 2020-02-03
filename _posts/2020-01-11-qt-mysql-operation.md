---
layout: post
title: "qt mysql 操作"
# subtitle: 'QT'
date:   2020-01-11 20:43:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: false
tags:
  - qt
  - technique
  - project
---
&emsp;在实验室项目过程中有需求自动存储数据到数据库，查了下mysql数据库提供了API供程序调用，这里总结下在C++程序中常用的api操作。

## c++ mysql操作
mysqlc.h文件：
```c++
#include <winsock.h>
#include <string>
#include <vector>
#include <mysql.h>

#pragma comment(lib, "libmysql.lib")

using namespace std;

class MySQLEngineC
{
public:
	MySQLEngineC(string host, string user, string passwd, string database, int port);
	~MySQLEngineC();
	void CreateTable(string tableName, vector<string>& fields);
	void InsertOneValue(string tableName, vector<double>& values);
	void InsertMultiValue(string tableName, vector<vector<double>>& values);
	void SelectData(string tableName);
	void CheckTable(string tableName);
	void DropTable(string tableName);
	void Close();
	MYSQL mydata;

private:
	int numFields;
};
```
mysqlc.cpp文件：
```c++
#include "mysqlenginec.h"
#include <string>

MySQLEngineC::MySQLEngineC(string host, string user, string passwd, string database, int port)
{
	if (mysql_library_init(0, NULL, NULL) != 0) {
		cout << "mysql_library_init() failed" << endl;
		return;
	}
	if (mysql_init(&mydata) == NULL) {
		cout << "mysql_init() failed" << endl;
		return;
	}
	if (mysql_options(&mydata, MYSQL_SET_CHARSET_NAME, "gbk") != 0) {
		cout << "mysql_options() failed" << endl;
		return;
	}
	if (mysql_real_connect(&mydata, host.c_str(), user.c_str(), passwd.c_str(), database.c_str(), port, NULL, 0) == NULL) {
		cout << "mysql_real_connect() failed" << endl;
		return;
	}
}

MySQLEngineC::~MySQLEngineC()
{

}

void MySQLEngineC::CreateTable(string tableName, vector<string>& fields)
{
	numFields = fields.size();
	string sqlstr;
	sqlstr = "CREATE TABLE IF NOT EXISTS " + tableName;
	sqlstr += "(";
	for (int i = 0; i < fields.size() - 1; ++i)
	{
		sqlstr += fields[i];
		sqlstr += " ";
		sqlstr += "DOUBLE(16, 10) NOT NULL,";
	}
	sqlstr += fields.back();
	sqlstr += " ";
	sqlstr += "DOUBLE(16, 10) NOT NULL";
	sqlstr += ");";
	cout << sqlstr << endl;
	if (mysql_query(&mydata, sqlstr.c_str()) != 0) {
		cout << "mysql_query() create table failed" << endl;
		return;
	}
	return;
}

void MySQLEngineC::InsertOneValue(string tableName, vector<double>& values)
{
	if (values.size() != numFields) {
		cout << "wrong values" << endl;
		return;
	}
	string sqlstr;
	sqlstr = "insert into " + tableName;
	sqlstr += " value";
	sqlstr += "(";
	for (int i = 0; i < values.size() - 1; ++i)
	{
		sqlstr += to_string(values[i]);
		sqlstr += ",";
	}
	sqlstr += to_string(values.back());
	sqlstr += ");";
	if (mysql_query(&mydata, sqlstr.c_str()) != 0) {
		cout << "mysql_query insert single value failed" << endl;
		return;
	}

}


void MySQLEngineC::InsertMultiValue(string tableName, vector<vector<double>>& values)
{
	for (int i = 0; i < values.size(); ++i)
	{
		InsertOneValue(tableName, values[i]);
	}
}

void MySQLEngineC::SelectData(string tableName)
{
	string sqlstr;
	sqlstr = "SELECT * FROM " + tableName;
	MYSQL_RES *result = NULL;
	if (mysql_query(&mydata, sqlstr.c_str()) != 0) {
		cout << "mysql_query() select data failed" << endl;
		mysql_close(&mydata);
		return;
	}
	result = mysql_store_result(&mydata);
	int rowCount = mysql_num_rows(result);
	cout << "row count:" << rowCount << endl;

	vector<string> title;
	vector<vector<double>> data;

	MYSQL_FIELD *field = NULL;
	unsigned int fieldCount = mysql_num_fields(result);
	for (unsigned int i = 0; i < fieldCount; ++i)
	{
		field = mysql_fetch_field_direct(result, i);
		cout << field->name << "\t\t";
		title.push_back(string(field->name));
	}

	MYSQL_ROW row = NULL;
	row = mysql_fetch_row(result);
	while (row != NULL) {
		vector<double> singleRow;
		for (int i = 0; i < fieldCount; ++i)
		{
			double t = stod(string(row[i]));
			cout << t << "\t\t";
			singleRow.push_back(t);
		}
		cout << endl;
		data.push_back(singleRow);
		row = mysql_fetch_row(result);
	}
	mysql_free_result(result);
}

void MySQLEngineC::CheckTable(string tableName)
{
	//查找刚才建的表
	string sqlstr = "SHOW TABLES LIKE '" + tableName + "'";
	if (mysql_query(&mydata, sqlstr.c_str()) == 0) {
		cout << "mysql_query() check table succeed" << endl;
	}
	else {
		cout << "mysql_query() check table failed" << endl;
	}
}

void MySQLEngineC::DropTable(string tableName)
{
	string sqlstr;
	sqlstr = "DROP TABLE " + tableName;
	if (mysql_query(&mydata, sqlstr.c_str()) != 0) {
		cout << "mysql_query() drop table failed" << endl;
		mysql_close(&mydata);
		return;
	}
	cout << "drop table " << tableName << " succeed." << endl;
}

void MySQLEngineC::Close()
{
	mysql_close(&mydata);
	mysql_server_end();
}
```

## qt mysql操作
&emsp;因为项目使用qt实现，而qt为各个数据库提供了统一的连接方式。这里实现的功能较上面少一些，不过够用，需要时再添加吧。  
mysqlengine.h文件
```c++
#include <iostream>
#include <string>
#include <vector>
#include <Qtsql>

using namespace std;

class MySQLEngine
{
public:
	MySQLEngine(string host, string user, string passwd, string database, int port);
	~MySQLEngine();
	void CreateTable(string tableName, vector<string>& fields);
	void InsertOneValue(string tableName, vector<double>& values);
	void InsertMultiValue(string tableName, vector<vector<double>>& values);
	/*void UpdateTable(string tableName, int row, int col, double val);*/
	void SelectData(string tableName, vector<vector<double>>& data, vector<string>& fields);
	void DropIfExist(string tableName);
	bool CheckTable(string tableName);
	void DropTable(string tableName);
	void Close();
	QSqlDatabase db;
	QSqlQuery *query;

private:
	int numFields;
};
```

mysqlengine.cpp文件
```c++
#include "mysqlengine.h"
#include <QDebug>
#include <QMessageBox>

MySQLEngine::MySQLEngine(string host, string user, string passwd, string database, int port)
{
	db = QSqlDatabase::addDatabase("QMYSQL");
	db.setHostName(QString::fromStdString(host));
	db.setUserName(QString::fromStdString(user));
	db.setPassword(QString::fromStdString(passwd));
	db.setDatabaseName(QString::fromStdString(database));
	db.setPort(port);

	if (!db.open()) {
		qDebug() << QObject::tr("Error:Open database failed");
		qDebug() << db.lastError();
	}
	else {
		qDebug() << "Open db success";
	}
	query = new QSqlQuery(db);
}

MySQLEngine::~MySQLEngine()
{

}

void MySQLEngine::CreateTable(string tableName, vector<string>& fields)
{
	numFields = fields.size();
	string sqlstr;
	sqlstr = "CREATE TABLE IF NOT EXISTS " + tableName;
	sqlstr += "(";
	for (int i = 0; i < fields.size() - 1; ++i)
	{
		sqlstr += fields[i];
		sqlstr += " ";
		sqlstr += "DOUBLE(16, 10) NOT NULL,";
	}
	sqlstr += fields.back();
	sqlstr += " ";
	sqlstr += "DOUBLE(16, 10) NOT NULL";
	sqlstr += ");";
	qDebug() << QString::fromStdString(sqlstr);

	query->clear();
	bool success = query->exec(QString::fromStdString(sqlstr));
	if (success) {
		qDebug() << QObject::tr("use table success.");
	}
	else {
		qDebug() << QObject::tr("use table failed.");
	}
}

void MySQLEngine::InsertOneValue(string tableName, vector<double>& values)
{
	if (!CheckTable(tableName)) {
		qDebug() << "table not exists!";
		return;
	}

	if (values.size() != numFields) {
		qDebug() << "wrong values";
		return;
	}
	string sqlstr;
	sqlstr = "insert into " + tableName;
	sqlstr += " value";
	sqlstr += "(";
	for (int i = 0; i < values.size() - 1; ++i)
	{
		sqlstr += to_string(values[i]);
		sqlstr += ",";
	}
	sqlstr += to_string(values.back());
	sqlstr += ");";
	// qDebug() << QString::fromStdString(sqlstr );

	query->clear();
	bool success = query->exec(QString::fromStdString(sqlstr));
	if (success) {
		//qDebug()<<QObject::tr("insert single to  table success.");
	}
	else {
		qDebug() << QObject::tr("insert single to  table failed.");
	}

}


void MySQLEngine::InsertMultiValue(string tableName, vector<vector<double>>& values)
{
	if (!CheckTable(tableName)) {
		qDebug() << "table not exists!";
		return;
	}
	for (int i = 0; i < values.size(); ++i)
	{
		InsertOneValue(tableName, values[i]);
	}
}

void MySQLEngine::SelectData(string tableName, vector<vector<double>>& data, vector<string>& fields)
{
	string sqlstr = "SELECT * FROM " + tableName;
	query->clear();
	fields.clear();
	bool success = query->exec(QString::fromStdString(sqlstr));
	while (query->next()) {
		QSqlRecord record = query->record();
		int colCount = record.count();
		if (fields.empty()) {
			for (int i = 0; i < colCount; ++i)
			{
				QSqlField column = record.field(i);
				fields.push_back(column.name().toStdString());
			}
		}
		vector<double> row;
		for (int i = 0; i < colCount; ++i)
			row.push_back(record.value(i).toString().toDouble());
		data.push_back(row);
	}
}

void MySQLEngine::DropIfExist(string tableName)
{
	string sqlstr = "DROP TABLE IF EXISTS " + tableName;
	query->clear();
	bool success = query->exec(QString::fromStdString(sqlstr));
}

bool MySQLEngine::CheckTable(string tableName)
{
	string sqlstr = "";
	sqlstr += "SHOW TABLES LIKE ";
	sqlstr += "'";
	sqlstr += tableName;
	sqlstr += "'";
	query->clear();
	bool success = query->exec(QString::fromStdString(sqlstr));
	if (success) {
		qDebug() << "check table exists";
		return 1;
	}
	else {
		qDebug() << "check table not exists";
		return 0;
	}
}

void MySQLEngine::DropTable(string tableName)
{
	string sqlstr = "";
	sqlstr += "DROP TABLE " + tableName;
	query->clear();
	bool success = query->exec(QString::fromStdString(sqlstr));
	if (success) {
		qDebug() << "drop table succeed";
	}
	else {
		qDebug() << "drop table failed";
	}
}

void MySQLEngine::Close()
{
	db.close();
}

```
&emsp;在qt调用mysql时常遇到驱动程序不正确，可以下载qt源码自己make一下，或者下载mysql connector c api，测试了下好像是c版本connector，不是c++版本connector。

## reference
[C/C++连接MySQL数据库例子](https://blog.csdn.net/to_Baidu/article/details/53819829)  
[Qt5.4下连接Mysql,QSqlDatabase: QMYSQL driver not loaded but available](https://blog.csdn.net/tenlee/article/details/43614241)  
[Qt的Mysql数据库表操作](https://blog.csdn.net/fanyun_01/article/details/53958275)  