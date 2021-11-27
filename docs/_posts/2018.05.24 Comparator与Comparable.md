---
title: Comparator与Comparable
date: 2018-05-24 14:57:12
tags: Java
---
# 先聊聊学习这个过程
学习分为主动和被动学习. 按着学校课程规定的来, 是被动学习, "学习"先入为主, 先人根据他们的经验, 给后人铺好路子, 构造了一条普适的高效的学习路线, 按部就班就能得到最好的效果. 如果学习者跟得上队伍, 那就会有所获. 主动学习是没有别人铺好的道路, 或者需要自己寻找前人铺好的路子. 这种学习过程一般是要经过: 自己的兴趣主导>自己学习>新的研究发现>和志同道合的人共同讨论 来达成.

# Comparable接口用在bean上
```
class Student implements Comparable<Student>{
    private String name;
    private int age;
    private float score;
	
	@Override
    public int compareTo(Student o) {
        // TODO Auto-generated method stub
        if(this.score>o.score)//score是private的，为什么能够直接调用,这是因为在Student类内部
            return -1;//由高到底排序
        else if(this.score<o.score)
            return 1;
        else{
            if(this.age>o.age)
                return 1;//由底到高排序
            else if(this.age<o.age)
                return -1;
            else
                return 0;
        }
    }
}
```
# Comparator用在集合外部
在设计类的时候，往往没有考虑到让类实现Comparable接口，那么我们就需要用到另外的一个比较器接口Comparator。
对于简单的比较,往往只需要实现Comparator内部类即可.
```
public class ComparableDemo02 {

    /**
     * @param args
     */
    public static void main(String[] args) {
        // TODO Auto-generated method stub

        Student stu[]={new Student("zhangsan",20,90.0f),
                new Student("lisi",22,90.0f),
                new Student("wangwu",20,99.0f),
                new Student("sunliu",22,100.0f)};
        java.util.Arrays.sort(stu,new StudentComparator());
        for(Student s:stu)
        {
            System.out.println(s);
        }
    }

}
```
