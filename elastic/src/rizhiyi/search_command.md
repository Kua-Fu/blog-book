# 搜索命令函数

| 函数| 描述| 示例|
| --- | --- | --- | 
| abs(X) | 此函数获取一个数字X，并返回其绝对值 | 以下示例返回absv，该变量的值为数值字段value的绝对值：…​ \| eval absv = abs(value) | 
| exp(X) | 此函数获取一个数字X，并返回e的X次方 | 以下示例返回y，该变量的值为e的3次方：…​ \| eval y = exp(3) | 
| ln(X)| 此函数获取一个数字X,并返回X的自然对数，以e 为 底| 以下示例返回y，该变量的值为bytes的自然对数：…​ \| eval lnBytes = ln(bytes)|
| empty(x) | 判断某个field是否为空, 也可写作 isempty(X) | empty(field) 如果存在返回false，否则返回true。也可写作isempty(field) 比如：empty(apache.status) |
| case(X, "Y", …​， [default, Z]) | 此函数会获取X, Y的参数对，X参数必须为布尔表达式，如果结果为true，则返回响应的Y的值，如果统计结果均为false, 则返回default对应的值，default部分为可选，不指定default，则default的返回为空值 | 以下示例返回http状态代码的描述 …​ \| eval desc = case(error == 200, "OK", error == 500, "Internal Server Error ", default, "Unexpected error") | 
| ceil(X) |  函数返回X向上取整的整数值 | 以下示例返回n = 5 …​ \| eval n = ceil(4.1) | 
| cidrmatch(X, Y)| 字段X必须是无分类和子网地址扩展(CIDR)，字段Y为一个IP地址，判断IP地址Y的子网地址是否和X匹配| 示例：<br> …​\| eval matched = cidrmatch("192.168.1.130/25", "192.168.1.129")' <br> 将192.168.1.130转换为二进制并保留高位的25位，低位设为0得到下限（不包括），对应的ip为192.168.1.128 <br> 将192.168.1.130转换为二进制保留高位的25位，低位全部设置为1得到上限（不包括），对应的ip地址为192.168.1.255 <br> 因此ip的范围是(192.168.1.128, 192.168.1.255) <br>  凡落在此范围的ip地址均match成功，因此matched的值为true <br> | 
| coalesce(X, …​)| 此函数接受任意数量的并返回第一个不为空值的值，如果所有参数都是空值，则返回空值 | 假设有一部分日志，用户名字段放在user_name或者user字段里，以下示例定义名为username的字段，该字段值为user_name和user字段不是空值的那一个： …​ \| eval username = coalesce(user_name, user) | 
| floor(X) | 函数向下取整为最接近的整数| 以下示例返回 n = 4 …​ \| eval n = floor(4.1) | 
| format(FORMAT, [X…​]) | 格式化字符串, 提供类似printf的功能 FORMAT，为printf函数的format字符串 | 示例：format("%.1fMB", rate)输出速率，rate保留小数点后一位, format("%s ⇒ %s", "aa", "bb")输入"aa ⇒ bb" , NOTE: 变量类型和format中%x需要对应正确，否则可能导致计算失败，而输出空值 | 
| formatdate(X[, Y]) | 该函数对X对应UTC时间值格式化为Y具体的时间格式; Y的时间格式字符串遵循java.text.SimpleDateFormat支持的格式，如果不指定Y，则默认的时间格式为"yyyy-MM-dd HH:mm:ss.SSS"，暂不支持时区的自定义 | 以下示例将返回timestamp所表示的时间的小时和分钟 …​ \| eval v = formatdate(timestamp, "HH:mm") | 
| datetime_diff(X, Y[, Z]) | 该函数接受两个时间戳，返回两个时间戳之间的时间差，单位为毫秒。可以指定返回的时间单位，d=天,h=小时,m=分钟,s=秒,默认为ms | 以下示例将返回1655870082000-1655870081000的时间差，单位为毫秒 …​ \| eval v = datetime_diff(1655870081000, 1655870082000) | 
| if(X, Y, Z) | 函数接受3个参数，第一个X为布尔表达式，如果X的计算结果为true，则结果为第二个参数Y的值，否则返回第三个参数Z值 | 以下示例将检查status的值，如果status==200，则返回”OK”，否则返回Error …​ \| eval desc = if (status == 200, "OK", "Error") | 
| in(field, X, …​)| 给定一个字段和若干指定值，判断字段中的值是否在指定值中存在。存在返回true，不存在返回false | 示例：…​ \| eval field = 'appname' \| where in(field, 'appname', 'hostname') | 
| isnum(X) | 判断字段X是否为数值类型，对于整数类型或者浮点型结果都会返回true，其它返回false | 示例： …​ \| eval a = isnum(apache.status) | 
| isstr(X) | 判断字段X是否为字符串类型| 示例: …​ \| eval a = isstr(apache.method) | 
| len(X) | 函数接收一个字符串类型的参数，返回字符串的长度 | 如果method的字段值为GET，以下示例n的值为3; …​ \| eval n = len(method) | 
| log(X [,Y]) | 此函数接受一个或两个数值类型的值，返回以Y为底X的对数，Y默认为10 | 以下示例将返回以2为底，number的对数 …​ \| eval num=log(number,2) | 
| pi() | 此函数返回pi的值 | 以下示例将返回圆的面积 …​ \| eval area_circle=pi()*pow(radius,2) | 
| pow(X, Y) | 此函数接受两个数值类型的参数，返回X的Y次方 | 假设number的值为2，以下示例将返回8 …​ \| eval n=pow(number,3) | 
| sqrt(X)| 此函数接受一个数值类型的参数，返回X的平方根 | 假设number的值为4，以下示例将返回2 …​ \| eval n=sqrt(number) | 
| acos(X) | 此函数接受一个范围在(-1,1)之间的数值类型的参数，返回以弧度表示的X的反余弦值 | 以下示例返回0的反余弦值 …​ \| eval result = acos(0) | 
| acosh(X) | 此函数接受一个大于等于1的数值类型的参数，返回以弧度表示的X的反双曲余弦值 | 以下示例返回1的反双曲余弦值 …​ \| eval result = acosh(1) | 
| asin(X) | 此函数接受一个范围在[-1,1]之间的数值类型的参数，返回以弧度表示的X的反正弦值 | 以下示例返回0的反正弦值 …​ \| eval result = asin(0) | 
| asinh(X) | 此函数接受一个数值类型的参数，返回以弧度表示的X的反双曲正弦值 | 以下示例返回5的反双曲正弦值 …​ \| eval result = asinh(5) |
| atan(X) | 此函数接受一个范围在 [-pi/2,+pi/2] 之间的数值类型的参数，返回以弧度表示的X的反正切值 | 以下示例返回0.5的反正切值 …​ \| eval result = atan(0.5) | 
| atan2(Y, X) | 此函数接受两个数值类型的参数，返回以弧度表示的 Y/X 的反正切值，X的取值范围在 [-pi,+pi]之间 | 以下示例返回0.5，0.75的反正切值 , …​ \| eval result = atan2(0.5, 0.75) | 
| atanh(X) | 此函数接受一个范围为(-1,1)之间的数值类型的参数，返回以弧度表示的X的反双曲正切值 | 以下示例返回0.5的反双曲正切值 …​ \| eval result = atanh(0.5) | 
| cos(X) | 此函数接受一个数值类型的参数，返回以弧度表示的X的余弦值 | 以下示例返回0的余弦值 …​ \| eval result = cos(0) | 
| cosh(X) | 此函数接受一个数值类型的参数，返回以弧度表示的X的双曲余弦值 | 以下示例返回0的双曲余弦值 …​ \| eval result = cosh(0) | 
| hypot(X, Y) | 此函数接受两个数值类型的参数，返回欧几里得范数，即sqrt(X^2 + Y^2) | 以下示例返回2，2的欧几里得范数 …​ \| eval result = hypot(2,2)  | 
| sin(X) | 此函数接受一个数值类型的参数，返回以弧度表示的X的正弦值 | 以下示例返回0的正弦值 …​ \| eval result = sin(0) | 
| sinh(X) | 此函数接受一个数值类型的参数，返回以弧度表示的X的双曲正弦值 | 以下示例返回0的双曲正弦值 …​ \| eval result = sinh(0) | 
|tan(X) | 此函数接受一个数值类型的参数，返回以弧度表示的X的正切值 | 以下示例返回0的正切值 …​ \| eval result = tan(0) |
| tanh(X) | 此函数接受一个数值类型的参数，返回以弧度表示的X的双曲正切值| 以下示例返回0的双曲正切值 …​ \| eval result = tanh(0) |
| entropy(field) | 此函数计算指定字段的熵值 | 以下示例返回json.name的熵值 …​ \| eval e = entropy(json.name) |
| lower(X) | 此函数接受一个字符串类型的参数，返回其小写形式 | 假设method的值为GET，以下示例将返回”get” …​ \| eval lowerstr = lower(method) |
| match(X, Y)| 此函数将使用正则表达式Y对X进行匹配，返回是否匹配成功 | 当且仅当字段于IP地址的基本形式匹配时，则返回true，否则返回false，这里使用了^和$表示执行完全匹配 …​ \| eval matched = match(ip, "^\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}$") |
| contains(X,Y)| 此函数会判断字符串X的值与字符串Y进行匹配，返回是否匹配成功,Y可以是多值字符串，Y是多值的情况下，包含Y中的任意一个则返回true，否则返回false| 以下实例将返回 true …​\|eval a = split("a,b,c,e",",")\|eval b = "Hello,world!" \|eval c = contains(b,a) |
| max(X,…​) | 此函数接受至少一个数值类型的参数，返回值较大的那个值 | 以下示例将返回101 …​ \| eval maxv = max(101, 100.0) |
| min(X,…​) | 此函数接受至少一个数值类型的参数，返回较小的那个值 | 以下示例将返回 100.0 …​ \| eval minv = min(101, 100.0) |
| mvsum(X,…​) | 此函数接受至少一个数值类型的多值字段或单值字段做为参数，返回所有参数的总和 | 以下示例将返回多值字段multiv的值与2的总和 …​ \| eval v = mvsum(multiv, 2) |
| mvavg(X,…​) | 此函数接受至少一个数值类型的多值字段或单值字段做为参数，返回所有参数的平均值| 以下示例将返回多值字段multiv的值与2的平均值,多值字段的每个值都会使计算平均值的分母加1 …​ \| eval v = mvavg(multiv, 2)|
| mvappend(X,…​)| 该函数为任意个参数，参数可以为字符串，多值字段，或者单值字段等| …​ \| eval v=mvappend(initv, "middle") |
|mvcount(X) | 该函数只有一个参数X，如果X是多值字段，则返回多值字段的值个数，如果是单值字段，则返回1，其他返回0| …​ \| eval c=mvcount(mvfield)|
|mvdedup(X)|该函数接收一个多值参数X，返回字段值消重后的多值类型| …​ \| eval v=mvdedup(mvfield)|
| mvfilter(X, filterexpr) | X为类型为多值的参数，filterexpr为过滤条件表达式，其中使用_x描述X中的单个值 对mv多值字段进行过滤，仅保留1a的值 | mvfilter(mv, _x == "1a")|
| mvfind(X,V) | X为多值类型的参数，V表示需要查找的值，如果找到返回对应下表，否则返回-1 | …​ \| eval n=mvfind(mymvfield, "err") |
| mvindex(X,start[, end]) | X为多值类型的参数，如果无end参数，则返回下表为start的元素，如果start不合法则返回null，否则返回从下标start到下标end（不包括）元素组成列表，如果下表范围不合法返回空数组， NOTE: 数组下表从0开始 | …​ \| eval v = mvindex(mv, 10, -1) |
|mvjoin(X,DELIMITER) | 将多值字段X的值使用分隔符DELEMITER组成一个字符串 | eval v = mvjoin(mv, ", ")| 
| mvmap(X,mapexpr) | X为多值类型，mapexpr为转换的表达式，使用_x表示X中的单个值，返回的多值类型为X中的每个元素使用mapexpr转换得到的值组成的多值类型 | …​ \| eval x = mvmap(X, tolong(_x) + 2) 设X = ["1", "3", "4"] 则x = [3, 5, 6] |
| mvdelta(X) | X为多值类型，该函数按顺序计算当前的值与前一个值的差,写入一个新的多值字段中，返回新的多值字段 | …​ \| eval x = mvdelta(X) 设X = ["1", "3", "4"] 则x = [2, 1] |
| mvrange(X,Y[,Z]) | 该函数使用一个数值的区间生成一个多值字段，其中X表示区间起始值，Y表示区间结束值（不包括），Z表示步跳数，默认为1 | 下例返回 1, 3, 5, 7. …​ \| eval mv=mvrange(1,8,2) |
| mvsort(X) | 对多值字段进行排序 | …​ \| eval s=mvsort(mv) | 
| mvszip(X,Y[,"Z"]) | X和Y都为多值类型，将X中的第一个元素和Y中的第一个元素都转换为字符串，并以Z为分隔符进行拼接，得到返回结果多值结果的第一个元素，类型为字符串，然后按照同样方法对X的第二个元素和Y中的第二个元素进行拼接，以此类推得到一个多值的结果。如果X和Y的长度不等，则当X或者Y处理完最后一个元素后不再进行拼接。 | X = [1, 3, 4, 7] Y = [2, 5, 8] mvszip(X, Y) = ["1,2", "3,5", "4,8"] |
| split(S, SEP) | X为字符串类型，使用字符串SEP分割符对S进行拆分成多值类型，如果SEP为空字符串，则S将被拆分为单字组成的多值类型 | 如X = ":abc::edf: " 则split(X, ":") = ["", "abc", "", "edf", " "] |
| now() | 函数用于表示当前时间，实际值为搜索请求收到的时间，在一个请求中多次调用返回的是同一个值，值为1970-01-01:00:00:00到当前时间的毫秒数，类型为long | 示例: …​\|eval current_time = now() |
| parsedate(X, Y[, Z]) | 解析日期时间串为unix时间戳 , X为日期的字符串，Y为日期的格式说明，遵循java.text.SimpleDateFormat支持的时间格式，Z为可选参数，指定Locale，默认为en（english） | 示例： parsedate("28/04/2016:12:01:01""dd/MM/yyyy:HH:mm:ss");parsedate("28/四月/2016", "dd/MMM/yyyy", "zh")其中zh表示中文的Locale;parsedate("2017-January-01", "yyyy-MMMM-dd", "UTC", "en")其中UTC代表时区，en表示英文的Locale |
| printf(FORMAT, [X…​])| 格式化字符串(格式) | 示例: printf("%.1fMB", rate)输出速率，rate保留小数点后一位 printf(“%s ⇒ %s", "aa", "bb")输入"aa ⇒ bb" NOTE: 变量类型和printf中%x需要对应正确，否则可能导致计算失败，⽽输出空值 |
| relative_time(X, Y) | 字段X必须是时间类型，字段Y必须为一个date math(请参考时间格式一节)的相对时间值，返回基于时间戳X的date math的计算结果 | 示例： …​ \| eval ts = relative_time(timestamp, "-1d/d") 返回得到timestamp所代表的时间减去1天的毫秒数，并圆整到0点，即timestamp表示的日期的前一天的零点。 |
| round(X [,Y]) | 把数值X近似到小数点后Y位;Y参数默认值为0，即近似为整数(round会把X近似为“最近的值”;当距离相同时，近似为更⼤的值) | 示例: round(3.14)输出3; round(3.1415,3)输出3.142; 注:当round(-1.5)输出-1;round(1.5)输出2 |
| substring(X, Y[, Z]) | 此函数接收三个参数，其中X必须为字符串，Y和Z是数字（Y和Z从0开始），返回X的子字符串，返回X的第Y个字符到第Z个（不包括）字符之间的字符 ，如果不指定Z 则返回Y位置开始的剩余字符串 | 以下示例返回"bce" …​ \| eval subs = substring("abcedfg", 1, 4) | 
| todouble(X) | 该函数接受一个参数，可以是字符串或者数值型，返回对应的双浮点数的值 | 以下示例返回123.1 …​ \| eval value = todouble("123.1") |
| tonumber | 将数字或字符串转化为数值类型，并可将2-36进制转换为10进制，base默认为10进制 | tonumber(numStr[,base] |
|tolong(X) | 该函数接受一个参数，字符串或者数值类型，返回对应的long值，X为10进制 | 以下示例返回123 …​ \| eval value=tolong("123") |
| tostring(X) | 该函数接收一个参数，可以是字符串或者数值类型，返回对应的字符串的值 | 以下示例返回”123.1” …​ \| eval strv = tostring(123.1) |
| tojson(X) | 该函数接收一个任意类型的参数，返回对应的 json 字符串 | 假设 a = [1,2,3],那么 "[1,2,3]" 是命令 …​\|eval json_str = tojson(a)的结果 |
| trim(X) | 该函数接受一个字符串类型的参数，返回X前后去除空白符的字符串值 |以下示例返回" bcd ef" …​ \| eval strv = trim(" bcd ef \t") |
| ltrim(X[, Y]) | 该函数接受一个字符串类型的参数，返回X去除左边空白符的字符串值。或者接收两个字符串类型的参数,用Y字符串从左侧去除在Y中出现过的字符,直到遇到第一个不在Y中的字符为止。 | 以下示例返回"yi" …​ \| eval strv = ltrim("rizhiyi", "irhz") |
|rtrim(X[, Y]) | 该函数接受一个字符串类型的参数，返回X去除右边空白符的字符串值。或者接收两个字符串类型的参数,用Y字符串从右侧去除在Y中出现过的字符,直到遇到第一个不在Y中的字符为止。| 以下示例返回"rizhiy" …​ \| eval strv = rtrim("rizhiyi", "irhz") |
| replace(<str>,<regex>,<replacement>) | 此函数用replacement字符串替换字符串str中每次出现的regex | 以下示例会将月份和日期数字调换位置。如果输入为 1/14/2020 ，则返回值为 14/1/2020。 …​ \| eval n=replace(date, "^(\d{1,2})/(\d{1,2})/", "\2/\1/") |
|typeof(X) | 获取字段X的类型 支持类型为: long, double, int, float, short, string, object, array, bool 如果字段为null， 则返回值是 null | 示例： …​ \| eval a_type = typeof(apache.method) |
| upper(X)| 函数接收一个字符串类型的参数，返回X的大写形式 | 以下示例返回GET …​ \| eval strv = upper("Get") | 
| urldecode(X) | 对字段X的值执行URL解码，字段X必须为字符串 NOTE: 目前还不支持指定字符编码 | 示例: …​ | eval url = urldecode(url) | 
| md5(X) | 对字段进行MD5编码 | 示例: …​ \| eval a = md5(X) |
| sha1(X) | 对字段进行SHA1编码 | 示例: …​ \| eval a = sha1(X) |
| sha256(X) | 对字段进行SHA256编码 | 示例: …​ \| eval a = sha256(X) |
| sha512(X) | 对字段进行SHA512编码 | 示例: …​ \| eval a = sha512(X)  |
| ip2long(X) | 将ip地址转化为long类型的数字 | 示例: …​ \| eval ipNum = ip2long(X) |
| long2ip(X) | 将long类型的数字转化为ip类型| 示例: …​ \| eval ip = long2ip(X) |
| cidr2long(X) | 将cidr路由转化为两个long类型的ip起止数字 | 示例: …​ \| eval ip_range = cidr2long(X) |
| is_valid_mac(X) | 判断是否为有效的mac地址，目前只支持六组冒号或横杠分隔的地址 | 示例: …​ \| eval is_valid_mac = is_valid_mac(X) |
| is_valid_ip(X) | 判断是否为有效的ip地址 | 示例: …​ \| eval is_valid_ip = is_valid_ip(X) |
| is_valid_mask(X) | 判断是否为有效的子网掩码，在[0, 32]之间 | 示例: …​ \| eval is_valid_mask = is_valid_mask(X) |
| is_valid_cidr(X) | 判断是否为有效的cidr地址 | 示例: …​ \| eval is_valid_cidr = is_valid_cidr(X) |
| expand_ip_range_to_cidr(X, Y [,Z]) | 将两个ip起止地址转化为cidr地址，X为ip起始地址，Y为ip终止地址，Z为可选参数cleanSingleIps，如果是true代表mask是32的cidr会去掉mask，否则不去掉，默认为false。| 示例: …​ \| eval cidr = expand_ip_range_to_cidr("192.168.1.1", "192.168.1.15") ; 或 …​ \| eval cidr = expand_ip_range_to_cidr("192.168.1.1", "192.168.1.15", true) …​ \| eval cidr = expand_ip_to_cidr(X) ; 或 …​ \| eval cidr = expand_ip_to_cidr(X, true) |
| like(X, Y) | 接收两个参数，X为一个字符串，Y是一个表达式。当X与Y匹配时，此函数返回true,否则返回false。表达式Y支持精确文本匹配，以及单字符_和多字符%匹配。 | 示例: …​ | eval is_like = like(X, "a%bc")|
| isnotnull(X) | 判断是否不为null，当x不为 null时候，返回true | 示例: …​ \| eval is_not_null = isnotnull(X) |
| isblank(X) | 判断是否为null或仅包含空白字符| 示例: …​ \| eval is_blank = isblank(X) |
| islong(X) | 判断是否为long类型字段 | 示例: …​ \| eval is_long = islong(X) |
| isbool(X) | 判断是否为boolean类型字段 | 示例: …​ \| eval is_bool = isbool(X) |
 

