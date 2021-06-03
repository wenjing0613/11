package week10;

import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class SizheTool {
    public static void main(String[] args) {
        try {
            System.out.println(jisuanStr("11.2+3.1*(423-(2+5.7*3.4)+5.6)-6.4/(15.5+24)"));
        } catch (Exception e) {
            System.out.println("请检查你的算式格式");
            e.printStackTrace();
        }
    }

    public static List<String> splitStr(String string) throws Exception {
        List<String> listSplit = new ArrayList<String>();
        Matcher matcher = Pattern.compile("\\-?\\d+(\\.\\d+)?|[*/()]|\\-")
                .matcher(string);// 用正则拆分成每个元素
        while (matcher.find()) {
            // System.out.println(matcher.group(0));
            listSplit.add(matcher.group(0));
        }
        return listSplit;
    }

    public static double jisuanStr(String str) throws Exception {
        double returnDouble = 0;
        List<String> listSplit = splitStr(str); // 拆分好的元素
        List<Integer> zKuohaoIdxList = new ArrayList<Integer>();// 左括号,<所在坐标，>
        if (Pattern.compile(".*\\(|\\).*").matcher(str).find()) {// 如果包含括号运算
            String value = "";// 单个字符值
            int zIdx = 0;// 上一个左括号在zKuoHaoIdxList的下标
            // 此层循环计算完所有括号里的算式
            List<String> tempList = new ArrayList<String>();// 前面没有计算的元素
            int removeL = 0;
            int tempListSize = 0;
            for (int i = 0; i < listSplit.size(); i++) {
                value = listSplit.get(i);
                tempList.add(value);
                tempListSize = tempList.size();
                if ("(".equals(value)) {// 左括号
                    zKuohaoIdxList.add(tempListSize-1);
                } else if (")".equals(value)) {// 遇到右括号就计算与上一左括号间的算式
                    zIdx = zKuohaoIdxList.size() - 1;// 离当前右括号最近的左括号配对
                    int start = zKuohaoIdxList.get(zIdx);
                    returnDouble = jisuan(tempList, start + 1, tempListSize-1); // 开始位置,就是上一个左括号
                    removeL = tempListSize - start;
                    tempList = removeUseList(tempList, removeL);// 移除已使用的元素
                    tempList.add(returnDouble + "");// 刚刚计算的值添加进来
                    zKuohaoIdxList.remove(zIdx);// 计算完毕清除括号
                }
            }
            // 把所有计算完
            returnDouble = jisuan(tempList, 0, tempList.size());
        } else {// 没有括号运算
            returnDouble = jisuan(listSplit, 0, listSplit.size());
        }
        return returnDouble;
    }

    public static List<String> removeUseList(List<String> list, int removeLength) {
        int le = list.size() - removeLength;
        for (int i = list.size() - 1; i >= le; i--) {
            list.remove(i);
        }
        return list;
    }

    public static double jisuan(List<String> listSplit, int start, int end)
            throws Exception {
        double returnValue = 0;
        String strValue = null;// 临时变量
        List<String> jjValueList = new ArrayList<String>();// 剩下的加减元素
        // 遍历计算乘除法
        for (int i = start; i < end; i++) {
            strValue = listSplit.get(i);
            if ("*".equals(strValue) || "/".equals(strValue)) {// 乘除
                strValue = jisuanValue("*".equals(strValue) ? "*" : "/", Double
                        .parseDouble(jjValueList.get(jjValueList.size() - 1)),
                        Double.parseDouble(listSplit.get(i + 1)))
                        + "";
                jjValueList.remove(jjValueList.size() - 1);
                i++;
            }
            jjValueList.add(strValue);
        }
        // 遍历计算加减
        for (int j = 0; j < jjValueList.size(); j++) {
            strValue = jjValueList.get(j);
            if ("-".equals(strValue) || "+".equals(strValue)) {
                returnValue = jisuanValue("-".equals(strValue) ? "-" : "+",
                        returnValue, Double.parseDouble(jjValueList.get(j + 1)));
                j++;
            } else {
                returnValue += Double.parseDouble(jjValueList.get(j));
            }
        }
        return returnValue;
    }

    public static double jisuanValue(String type, double start, double end)
            throws Exception {
        double d = 0;
        if ("-".equals(type)) {
            d = start - end;
        } else if ("+".equals(type)) {
            d = start + end;
        } else if ("*".equals(type)) {
            d = start * end;
        } else if ("/".equals(type)) {
            if (0 == start || 0 == end)
                d = 0;
            else
                d = start / end;
        }
        return d;
    }

}

