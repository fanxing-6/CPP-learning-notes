# `boost`序列化

## 字符串序列化文本文件

```cpp
void save_string_to_txt()
{
	std::ofstream file("a.txt");
	boost::archive::text_oarchive oa(file);
	string s = "hello world!";
	oa << s;
}
void load_string_from_txt()
{
	std::ifstream file("a.txt");
	boost::archive::text_iarchive ia(file);
	string s;
	ia >> s;
	cout << s << endl;
}
```

## 字符串序列化`xml`

```cpp
```

