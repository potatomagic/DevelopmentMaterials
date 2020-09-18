获得月份天数
```java
   /**
     * 通过年份和月份 得到当月的日数
     */
    public static int getMonthDays(int year, int month) {
        month++;
        switch (month) {
            case 1:
            case 3:
            case 5:
            case 7:
            case 8:
            case 10:
            case 12:
                return 31;
            case 4:
            case 6:
            case 9:
            case 11:
                return 30;
            case 2:
                if (((year % 4 == 0) && (year % 100 != 0)) || (year % 400 == 0)) {
                    return 29;
                } else {
                    return 28;
                }
            default:
                return -1;
        }
    }
```

两个日期相差几周
```java
   /**
     * 两个日期距离几周
     */
    public static int getWeeksNum(int lastYear, int lastMonth, int lastDay, int year, int month, int day) {
        int startDayOfWeek = 0;
        try {
            startDayOfWeek = dayForWeek(lastYear + "-" + lastMonth + "-" + lastDay);
        } catch (Exception e) {
            e.printStackTrace();
        }
        int endDayOfWeek = 0;
        try {
            endDayOfWeek = dayForWeek(year + "-" + month + "-" + day);
        } catch (Exception e) {
            e.printStackTrace();
        }
        int startDayOfYear = 0;
        try {
            startDayOfYear = dayForYear(lastYear + "-" + lastMonth + "-" + lastDay);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        int endDayOfYear = 0;
        try {
            endDayOfYear = dayForYear(year + "-" + month + "-" + day);
        } catch (ParseException e) {
            e.printStackTrace();
        }

        int dayNum;
        if (lastYear == year) {
            dayNum = (endDayOfYear - endDayOfWeek) - (startDayOfYear - startDayOfWeek);
        } else if (lastYear < year) {
            if ((year - lastYear) == 1) {
                dayNum = ((getMonthDays(lastYear, 2) + 337)
                        - (startDayOfYear - startDayOfWeek))
                        + (endDayOfYear - endDayOfWeek);
            } else {
                dayNum = ((getMonthDays(lastYear, 2) + 337) - (startDayOfYear - startDayOfWeek))
                        + (getMonthDays(lastYear + 1, 2) + 337)
                        + (endDayOfYear - endDayOfWeek);
            }
        } else {
            if ((lastYear - year) == 1) {
                dayNum = ((getMonthDays(year, 2) + 337)
                        - (endDayOfYear - endDayOfWeek))
                        + (startDayOfYear - startDayOfWeek);
            } else {
                dayNum = ((getMonthDays(year, 2) + 337) - (startDayOfYear - startDayOfWeek))
                        + (getMonthDays(year + 1, 2) + 337)
                        + (startDayOfYear - startDayOfWeek);
            }
        }
        int weekNum = dayNum / 7;

        return weekNum;
    }
```

获得该日期在一周中第几天
```java
    /**
     * 获得该日期在一周中第几天
     * 日：1 一：2  二：3  三：4 四：5  五：6 六：7
     */
    public static int dayForWeek(String time) throws Exception {
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        Calendar c = Calendar.getInstance();
        c.setTime(format.parse(time));
        int dayForWeek = c.get(Calendar.DAY_OF_WEEK);
        return dayForWeek;
    }
```

获得该日期在一年中第几天
```java
    /**
     * 获得该日期在一年中第几天
     */
    public static int dayForYear(String time) throws ParseException {
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        Calendar c = Calendar.getInstance();
        c.setTime(format.parse(time));
        int dayForYear = c.get(Calendar.DAY_OF_YEAR);
        return dayForYear;
    }
```

根据国历获取假期
```java
    /**
     * 根据国历获取假期
     */
    public static String getHolidayFromSolar(int year, int month, int day) {
        String message = "";
        if (month == 0 && day == 1) {
            message = "元旦";
        } else if (month == 1 && day == 14) {
            message = "情人节";
        } else if (month == 2 && day == 8) {
            message = "妇女节";
        } else if (month == 2 && day == 12) {
            message = "植树节";
        } else if (month == 3) {
            if (day == 1) {
                message = "愚人节";
            } else if (day >= 4 && day <= 6) {
                if (year <= 1999) {
                    int compare = (int) (((year - 1900) * 0.2422 + 5.59) - ((year - 1900) / 4));
                    if (compare == day) {
                        message = "清明节";
                    }
                } else {
                    int compare = (int) (((year - 2000) * 0.2422 + 4.81) - ((year - 2000) / 4));
                    if (compare == day) {
                        message = "清明节";
                    }
                }
            }
        } else if (month == 4 && day == 1) {
            message = "劳动节";
        } else if (month == 4 && day == 4) {
            message = "青年节";
        } else if (month == 4 && day == 12) {
            message = "护士节";
        } else if (month == 5 && day == 1) {
            message = "儿童节";
        } else if (month == 6 && day == 1) {
            message = "建党节";
        } else if (month == 7 && day == 1) {
            message = "建军节";
        } else if (month == 8 && day == 10) {
            message = "教师节";
        } else if (month == 9 && day == 1) {
            message = "国庆节";
        } else if (month == 10 && day == 11) {
            message = "光棍节";
        } else if (month == 11 && day == 25) {
            message = "圣诞节";
        }
        return message;
    }
```

将时间戳转换为日期
```java
    /**
      * 将时间戳转换为日期
      */
    public static String stampToDate(long s) {
        String res;
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        long lt = new Long(s);
        Date date = new Date(lt);
        res = simpleDateFormat.format(date);
        String[] select = res.split("-");
        int month = Integer.valueOf(select[1]) - 1;
        String monthStr = (month < 10) ? "0" + month : month + "";
        res = select[0] + "-" + monthStr + "-" + select[2];
        return res;
    }
```
