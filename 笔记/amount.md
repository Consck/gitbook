## 将金额转换为大写
```Java
public static String digitCapital1(String nStr) {
        String s = "";
        double amount = Double.parseDouble(nStr);
        // 小数部分
        String[] split = nStr.split("\\.");
        if (split.length > 1) {
            // 小数点为特殊符号，在分割时需进行转义
            String decimalStr = split[1];
            while(decimalStr.charAt(decimalStr.length()-1) == '0'){
                if(decimalStr.length()-1 <= 0){
                    break;
                }
                decimalStr = decimalStr.substring(0,decimalStr.length()-1);
            }
            if (decimalStr.length() > 2) {
                decimalStr = decimalStr.substring(0, 2);
            }
            // 将小数部分转换为整数
            Integer integer = Integer.valueOf(decimalStr);
            String p = "";
            for (int i = 0; i < decimalStr.length() ; i++) {
                p = digit[integer % 10]  + p;
                integer = integer / 10;
            }
            if(!"零".equals(p)){
                s = "点" + p  + s;
            }
        }
        int integerPart = (int)Math.floor(amount);
        if(integerPart == 0){
            s = digit[integerPart] + s;
        }
        // 整数部分
        for (int i = 0; i < unit[0].length && integerPart > 0; i++) {
            String p = "";
            for (int j = 0; j < unit[1].length && integerPart > 0; j++) {
                p = digit[integerPart % 10] + unit[1][j] + p;
                integerPart = integerPart / 10;
            }
            s = p.replaceAll("(零.)+", "零").replaceAll("(零.)*零$", "").replaceAll("^$", "零") + unit[0][i] + s;
        }
        if(s.startsWith("壹拾")){
            s = s.replace("壹拾", "拾");
        }
        if(s.contains("点")){
            String[] str = s.split("点");
            str[0] = str[0].replaceAll("零拾","").replaceAll("零佰","")
                    .replaceAll("零仟","").replaceAll("零万","").replaceAll("零亿","");
            s = str[0] + "点" + str[1];
            if(!s.startsWith("零点")){
                s = s.replaceAll("零点", "点");
            }
        }
        while(s.charAt(s.length()-1) == '零'){
            if(s.length()-1 <= 0){
                break;
            }
            s = s.substring(0, s.length() - 1);
        }
        return s;
    }
```
## 验证身份证
```Java
public static boolean checkIdCardNum(String idNum) {
        idNum=idNum.toUpperCase();
        String regex="";
        regex+="^[1-6]\\d{5}";
        regex+="(18|19|20)\\d{2}";
        regex+="((0[1-9])|(1[0-2]))";
        regex+="(([0-2][1-9])|10|20|30|31)";
        regex+="\\d{3}";
        regex+="[0-9X]";

        if(!idNum.matches(regex)) {
            return false;
        }

        //第1，2位(省)打表进一步校验
        int[] d={11,12,13,14,15,
                21,22,23,31,32,33,34,35,36,37,
                41,42,43,
                44,45,46,
                50,51,52,53,53,
                61,62,63,64,65,
                83,81,82};
        boolean flag=false;
        int prov=Integer.parseInt(idNum.substring(0, 2));
        for(int i=0;i<d.length;i++) {
            if(d[i]==prov)
            {
                flag=true;
                break;
            }
        }
        if(!flag) {
            return false;
        }

        //生日校验：生日的时间不能比当前时间（指程序检测用户输入的身份证号码的时候）晚
        SimpleDateFormat sdf=new SimpleDateFormat("yyyyMMdd");
        try{
            Date birthDate=sdf.parse(idNum.substring(6, 14));
            Date curDate=new Date();
            if(birthDate.getTime()>curDate.getTime()) {
                return false;
            }
        }catch (Exception ex){
            LogUtil.error(log,"日期解析失败, idNum = {}",idNum);
        }
        //生日校验：每个月的天数不一样（有的月份没有31），还要注意闰年的二月
        int year=Integer.parseInt(idNum.substring(6, 10));
        int leap=((year%4==0 && year%100!=0) || year%400==0)?1:0;
        final int[] month={0,31,28+leap,31,30,31,30,31,31,30,31,30,31};
        int mon=Integer.parseInt(idNum.substring(10, 12));
        int day=Integer.parseInt(idNum.substring(12, 14));
        if(day>month[mon]) {
            return false;
        }
        //检验码
        if(idNum.charAt(17)!=getLastChar(idNum)) {
            return false;
        }
        return true;
    }

    /**
     * 根据身份证号码的前17位计算校验码
     * @param idNum
     * @return
     */
    private static char getLastChar(String idNum)
    {
        final int[] w = {0, 7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2};
        final char[] ch = {'1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2'};
        int res = 0;
        for (int i = 0; i < 17; i++) {
            int t = idNum.charAt(i) - '0';
            res += (t * w[i + 1]);
        }
        return ch[res % 11];
    }
```

## base64编解码
```Java
final Base64.Decoder decoder = Base64.getDecoder();
final Base64.Encoder encoder = Base64.getEncoder();
final String text = "字串文字";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = encoder.encodeToString(textByte);
System.out.println(encodedText);
//解码
System.out.println(new String(decoder.decode(encodedText), "UTF-8"));

final Base64.Decoder decoder = Base64.getDecoder();
final Base64.Encoder encoder = Base64.getEncoder();
final String text = "字串文字";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = encoder.encodeToString(textByte);
System.out.println(encodedText);
//解码
System.out.println(new String(decoder.decode(encodedText), "UTF-8"));
```
## 字节数组&字符串
```Java
byte [] b =s.getBytes();
String s = new String(b);
```