# CellularNetwork
- 代工的课设

# 蜂窝数字移动通信的简单模拟
##面向对象程序设计大作业
[toc]
##1. 需求分析
###1.1 问题描述
- 蜂窝数字移动通信系统的工作原理如下：
    1. 在覆盖区域内分布一定数量的通信基站，基站之间的距离小于等于常量R，每个基站覆盖范围为以R为半径的圆形区域；
    2. 任意基站之间可以直接通信，手机总是和距离最近的基站通信，手机之间不能直接通信；
    3. 手机进入某基站覆盖范围内，向基站注册自身信息（报到），机按一定时间间隔向所在的基站发送心跳信号；
    4. 当一部手机与另外一部手机通信时，首先将拨号指令发送给所在基站，所在基站接收到拨号指令后，询问其他基站，是否有被叫手机。被叫手机所在基站告知主叫手机所在基站手机在此基站范围，主叫手机基站向被叫手机基站发送拨号指令，被叫手机基站将拨号指令发送给被叫手机，被叫手机接听，完成通话。
- 请大家并用Java语言实现一套模拟蜂窝数字移动通信的仿真程序，要求：
    1. 程序开始时，基站位置不变，手机位置随机移动，区域内手机数量不定，手机随机发起通话。
    2. 可以打，可以接；
    3. 可以处理通信过程中手机移动变换基站的情况（选做）；
    4. 通过打印信息表示通信过程，不必处理语音等。
- 程序以命令行的形式运行即可，不需要编写图形界面，也不需要接受命令行输入。程序应充分体现面向对象的思想。

###1.2 对问题的理解
- 从题目可以得知，我们需要实现一个区域的表示，其中，一些坐标上有固定的基站，部分坐标上分布有手机；
- 每个手机可以随机移动，可以随机拨打电话、挂断电话；这样就产生了拨打电话的很多种情况：对方占线或没在服务区，同理，挂断也要考虑非人为因素挂断，如离开基站；
- 手机随机移动的时候可能也正在通话，也要考虑基站转移、或者进入无服务区挂断的情况；
- 可以把地图可视化的打印出来，更加直观。


##2. 程序设计
###2.1 概要设计
- 首先，需要设计一个生成随机数的接口 **Interface RandomN** ，其中，需要有可以实现生成 [L, R] 之间的随机数的方法 **randomNumber()**， 以及随机生成 true/false 的随机选择函数 **randomChoice()**。由于之后很多函数、类的设计中都需要用到随机操作，所以这个接口的设计很有必要。
- 其次，需要设计四个类，分别为：
    - **Class BaseStation** 此类为基站类，一个基站的属性有：
        -  **int stationID** (基站序号)；
        -  **int position_x/position_y** (基站的坐标)；
        -  **int area_R** (基站作用范围)。

        
    - **Class Phone** 此类为手机类，一个手机的属性有 ：
        - **int phoneID** (手机的ID)；
        - **int cphoneID** (如果该手机在通话中，则此为与之通话的手机的ID；否则为-1) ；
        - **Stringnumber** (手机号码)；
        - **int position_x/position_y** (手机的坐标)；
        - **int state** 手机的状态)；
        - **int in_stationID** (如果手机位于某个基站范围内，则为该基站ID；否则为-1)。
 

    - **Class ComMap** 此类为地图类，主要实现可视化地图、手机随机操作等功能，其属性有：
        - **int width/height** (地图的宽高)；
        - **int R** (基站作用范围)；
        - **int phonenum** (手机的个数)；
        - **char[][] map** (可视化地图)；
        - **Arraylist<BaseStation> baseStations** (所有基站)；
        - **Arraylist<Phone> phones** (所有手机)；
    - **Class Main** 此类为主程序。
- 概要的流程大概是：
    1. 初始化各个参数，构造一个 ComMap 对象 comMap，构造函数会打印出当前地图信息(地图标注了基站的位置，基站在地图中的 Symbol 从字符 'a' 开始递增，即 基站1 对应图中字符为 'a'，基站2 对应图中字符为 'b'等；同时标注了手机的位置，同理手机的 Symbol 从字符 '0' 开始递增。需要注意的是，如果手机恰好位于基站的位置，则仅显示手机的 Symbol)；
    2. 调用 comMap.refreshMap() 函数，该函数首先会重新载入地图，填充基站位置信息，再对每部手机随机进行移动，打印出各个手机对应的基站信息(可能无信号)，再向地图填充手机位置信息；
    3. 该函数之后会遍历每个手机，根据每个手机的位置、通话状态随机进行拨号、挂断或者无信号导致通话挂断、跨基站通话等操作，并进行打印输出；
    4. 打印新的地图；
    5. 循环调用 comMap.refreshMap() 函数。

###2.2 详细设计
- 以下具体介绍每个类中重要的方法及功能，注意，下述并非代码，仅是对详细的代码流程作说明：
- **Interface RandomN 主要接口:**
    ```JAVA
    //生成[L, R]的随机整数
    public int randomNumber(int l, int r) 
    //生成随机boolean: true/false
    public boolean randomChoice()
    ```
    
- **Class BaseStation 主要方法:** 
    ```JAVA
    //由于基站是静态的，故该类没有特殊的方法，仅有普通的get/set属性的函数，不多做介绍。
    ```
        
- **Class Phone 主要方法:**
    ```JAVA
    //构造函数，
    Phone(int phoneID, int width, int height){

        int num = randomNumber(100000000,999999999);
        this.phoneID = phoneID;
        this.number = "18" + num;   //随机生成number
        this.state = 0;             //处于空闲状态
        this.position_x = randomNumber(0, width);   //随机生成x
        this.position_y = randomNumber(0, height);  //随机生成y
        this.width = width;
        this.height = height;
        this.in_stationID = -1;     //初始状态设置为无基站覆盖

    }
    //沿x轴方向随机正反步进一格
    public void move_x(boolean mo)
    //沿y轴方向随机正反步进一格
    public void move_y(boolean mo)
    ```
 
- **Class ComMap 主要方法:** 
    ```JAVA
    //检查离手机最近的基站，返回id，失败返回-1
    public int checkStation(Phone phone) 
    //构造函数
    ComMap(int width, int height, int R, int num, int phonenum, ArrayList<Phone> phones){
        //随机生成基站并加入ArrayList，填充map
        //读取手机，填充map,并查询其进入的基站报道
        printMap();
    }
    public void checkPhone(Phone phone){

        //生成随机操作对象cPhone，可能会向他拨号
        //正在通话中的话
        if (phone.getState() == 1) {
            //phone通话中的对象oldcPhone
            //如果手机1不在服务区
            if (phone.getIn_stationID() == -1) {
                //通话失败，打印信息，原因是手机1不在服务区，状态恢复
                phone.setState(0);
                oldcPhone.setState(0);
            }
            else {
                //在服务区但正在通话的对象不在服务区
                if (oldcPhone.getIn_stationID() == -1){
                    //通话失败，打印信息，原因是手机2不在服务区，状态恢复
                    phone.setState(0);
                    oldcPhone.setState(0);
                }
                else {
                    //都在服务区，但是本机可能随机挂断或者持续通话
                    if (randomChoice()) {
                        //手机1挂断了电话，状态恢复，打印信息
                        phone.setState(0);
                        oldcPhone.setState(0);
                    } else {
                        //没有挂断，打印信息，显示仍然在通话中，打印数据通信经过的基站信息
                    }
                }
            }
        }
        //处于空闲状态的话
        else {
            //如果在服务区
            if (phone.getIn_stationID() != -1) {
                if (randomChoice()) {
                    //50%的概率随机尝试拨打电话
                    //拨打对象不在服务区
                    if (cPhone.getIn_stationID() == -1) {
                        //呼叫失败，打印呼叫指令通过的基站传输链
                    }
                    //拨打对象处于通话中
                    if (cPhone.getState() == 1) {
                        //呼叫失败，打印呼叫指令通过的基站传输链
                        return;
                    }
                    //否则通话成功，更新信息，打印呼叫指令通过的基站传输链
                    phone.setState(1);
                    phone.setCphoneID(cPhone.getPhoneID());
                    cPhone.setState(1);
                    cPhone.setCphoneID(phone.getPhoneID());
                }
                //50%的概率并不拨打电话
                else{
                }
            }
            //如果本机没在服务区，失败，打印信息
            else{
            }
        }

    }
    //打印地图
    public void printMap()
    //刷新地图(随机移动，并让手机进行随机动作)
    public void refreshMap() {
        //重载地图
        //让手机随机移动
        for (Phone phone : phones) {
            phone.move_x(randomChoice());
            phone.move_y(randomChoice());
            map[phone.getPosition_x()][phone.getPosition_y()] = (char) (phone.getPhoneID() + '0');
        }
        //调用 checkPhone() 依据状态随机拨号、挂断等
        for (Phone phone : phones) {
            checkPhone(phone);
        }
        //打印新地图
        printMap();
    }
    ```    
    
     
- **Class Main** 此类为主程序。
     ```JAVA
    public static void main(String[] args) {
        //参数设置
        int width = ;     //地图宽度
        int height = ;    //地图高度
        int R = ;          //基站信号半径
        int baseStation_num = ;   //基站数量
        int phone_num = ;  //手机数量
        int times = ;     //地图刷新次数（手机随机移动、随机拨号、随机挂断）
        ArrayList<Phone> phones = new ArrayList<>();

        //生成手机
        //生成地图(含基站)
        //循环刷新地图
        for (i = 0; i < times; i ++) comMap.refreshMap();
    }

    ```

##3. 代码清单
- 运行环境：
    - OS: Windows 10
    - IDE: Intellij IDEA
- **BaseStation.java**
    ```JAVA
        //基站类
    
    public class BaseStation {
    
        private int stationID;
        private int position_x;
        private int position_y;
        private int area_R;
    
        BaseStation(int stationID, int position_x, int position_y, int area_R){
            this.stationID = stationID;
            this.position_x = position_x;
            this.position_y = position_y;
            this.area_R = area_R;
        }
    
        public void setPosition_x(int position_x) {
            this.position_x = position_x;
        }
        public int getPosition_x() {
            return position_x;
        }
        public void setPosition_y(int position_y) {
            this.position_y = position_y;
        }
        public int getPosition_y() {
            return position_y;
        }
        public void setStationID(int stationID) {
            this.stationID = stationID;
        }
        public int getStationID() {
            return stationID;
        }
        public void setArea_R(int area_R) {
            this.area_R = area_R;
        }
        public int getArea_R() {
            return area_R;
        }
    
    }
    ```
    
    
- **Phone.java**
    ```JAVA
        //手机类
    
    public class Phone implements RandomN{
    
        private final int phoneID;ååç
        private int cphoneID;
        private final String number;
        private int position_x;
        private int position_y;
        private int state;
        private final int width;
        private final int height;
        private int in_stationID;
    
        Phone(int phoneID, int width, int height){
    
            int num = randomNumber(100000000,999999999);
            this.phoneID = phoneID;
            this.number = "18" + num;
            this.state = 0;
            this.position_x = randomNumber(0, width);
            this.position_y = randomNumber(0, height);
            this.width = width;
            this.height = height;
            this.in_stationID = -1;
    
        }
    
        public int getPhoneID() {
            return phoneID;
        }
        public void setCphoneID(int cphoneID) {
            this.cphoneID = cphoneID;
        }
        public int getCphoneID() {
            return cphoneID;
        }
        public void setPosition_x(int position_x) {
            this.position_x = position_x;
        }
        public int getPosition_x() {
            return position_x;
        }
        public void setPosition_y(int position_y) {
            this.position_y = position_y;
        }
        public int getPosition_y() {
            return position_y;
        }
        public String getNumber() {
            return number;
        }
        public void setState(int state) {
            this.state = state;
        }
        public void setIn_stationID(int in_stationID) {
            this.in_stationID = in_stationID;
        }
        public int getIn_stationID() {
            return in_stationID;
        }
        public int getState() {
            return state;
        }
    
        public void move_x(boolean mo) {
            int newx;
            if (mo) {
                newx = getPosition_x() + 1;
                if (newx < width)   setPosition_x(getPosition_x() + 1);
                else                setPosition_x(getPosition_x() - 1);
            } else {
                newx = getPosition_x() - 1;
                if (newx >= 0)      setPosition_x(getPosition_x() - 1);
                else                setPosition_x(getPosition_x() + 1);
            }
        }
        public void move_y(boolean mo){
            int newy;
            if (mo) {
                newy = getPosition_y() + 1;
                if (newy < height)   setPosition_y(getPosition_y() + 1);
                else                setPosition_y(getPosition_y() - 1);
            } else {
                newy = getPosition_y() - 1;
                if (newy >= 0)      setPosition_y(getPosition_y() - 1);
                else                setPosition_y(getPosition_y() + 1);
            }
        }
    
        @Override
        public int randomNumber(int l, int r) {
            return (int)(Math.random() * (r + 1 - l) + l);
        }
    
        @Override
        public boolean randomChoice() {
            double r = Math.random();
            return r < 0.5;
        }
    
    }

    ```
    
- **RandomN.java**
    ```JAVA
    public interface RandomN {
    //生成[l, r]随机数
    public int randomNumber(int l, int r);
    //主要用于二选一的随机操作(50%概率)
    public boolean randomChoice();
    }


    ```
    
- **ComMap.java**
    ```JAVA
    import java.util.ArrayList;
    
    public class ComMap implements RandomN{
    
        //地图的基本参数
        final private int width;
        final private int height;
        final private int R;
        final private int phonenum;
        final char[][] map = new char[10000][10000];
        ArrayList<BaseStation> baseStations;
        ArrayList<Phone> phones;
    
        //初始化
        ComMap(int width, int height, int R, int num, int phonenum, ArrayList<Phone> phones){
    
            this.width = width;
            this.height = height;
            this.R = R;
            this.phonenum = phonenum;
            this.phones = phones;
    
            baseStations = new ArrayList<>();
    
            int i, j;
            for (i = 0; i < width; i ++) for (j = 0; j < height; j ++)
                map[i][j] = '-';
            for (i = 0; i < num; i ++){
                int x = randomNumber(0, getWidth());
                int y = randomNumber(0, getHeight());
                BaseStation baseStation = new BaseStation(i+1, x, y, R);
                map[x][y] = (char)(baseStation.getStationID() + 'a');
                baseStations.add(baseStation);
            }
    
            for (Phone phone : phones) {
                map[phone.getPosition_x()][phone.getPosition_y()] = (char) (phone.getPhoneID() + '0');
                int id = checkStation(phone);
                if (id >= 0) System.out.println("phone_" + phone.getPhoneID() + " " + phone.getNumber() +
                        " has entered the " + id + " base station." + " Station Sysmbol in map: " + (char) (id + 'a'));
                else System.out.println("phone_" + phone.getPhoneID() + " " + phone.getNumber() + " is out of service.");
            }
            System.out.println();
            printMap();
            System.out.println();
    
        }
    
        public int getR() {
            return R;
        }
        public int getWidth() {
            return width;
        }
        public int getHeight() {
            return height;
        }
        public char[][] getMap() {
            return map;
        }
        public int getPhonenum() {
            return phonenum;
        }
    
        //刷新地图(随机移动，并让手机进行随机动作)
        public void refreshMap() {
            //重载地图，并让手机随机移动
            int i, j;
            for (i = 0; i < width; i ++) for (j = 0; j < height; j ++)
                map[i][j] = '-';
            for (BaseStation baseStation : baseStations) {
                map[baseStation.getPosition_x()][baseStation.getPosition_y()] = (char) (baseStation.getStationID() + 'a');
            }
            for (Phone phone : phones) {
                phone.move_x(randomChoice());
                phone.move_y(randomChoice());
                map[phone.getPosition_x()][phone.getPosition_y()] = (char) (phone.getPhoneID() + '0');
    
                int id = checkStation(phone);
                if (id >= 0) System.out.println("phone_" + phone.getPhoneID() + " " + phone.getNumber() +
                        " has entered the " + id + " base station." + " Station Sysmbol in map: " + (char) (id + 'a'));
                else System.out.println("phone_" + phone.getPhoneID() + " " + phone.getNumber() + " is out of service.");
    
            }
    
            //随机拨号、挂断等
            System.out.println();
            for (Phone phone : phones) {
                checkPhone(phone);
                System.out.println();
            }
            printMap();
            System.out.println();
        }
    
        //手机进行随机动作
        public void checkPhone(Phone phone){
    
            //生成随机操作对象cPhone，可能会向他拨号
            int cphone = randomNumber(0, getPhonenum());
            while (cphone == phone.getPhoneID()){
                cphone = randomNumber(0, getPhonenum());
            }
    
    //        System.out.println(cphone);
            //正在通话中的话
            if (phone.getState() == 1) {
                //phone通话中的对象oldcPhone
                Phone oldcPhone = phones.get(phone.getCphoneID());
                //如果手机1不在服务区
                if (phone.getIn_stationID() == -1) {
    
                    System.out.println("Because " + "phone_" + phone.getPhoneID() + "(" + phone.getNumber() + ")" +
                            " is out of service. The call with phone_" + oldcPhone.getPhoneID() + " has been terminated.");
                    System.out.println(phone.getPhoneID() + "--" + "NO BASESTATION"
                            + "||" + (char) (oldcPhone.getIn_stationID() + 'a')+ "--" + oldcPhone.getPhoneID());
                    phone.setState(0);
                    oldcPhone.setState(0);
                }
                else {
                    //在服务区但正在通话的对象不在服务区
                    if (oldcPhone.getIn_stationID() == -1){
                        System.out.println("Because " + "phone_" + oldcPhone.getPhoneID() + "(" + phone.getNumber() + ")" +
                                " is out of service. The call with phone_" + phone.getPhoneID() + " has been terminated.");
                        System.out.println(oldcPhone.getPhoneID() + "--" + "NO BASESTATION"
                                + "||" + (char) (phone.getIn_stationID() + 'a')+ "--" + oldcPhone.getPhoneID());
                        phone.setState(0);
                        oldcPhone.setState(0);
                    }
                    else {
                        //都在服务区，但是本机可能随机挂断或者持续通话
                        if (randomChoice()) {
                            System.out.println("Because " + "phone_" + phone.getPhoneID() + "(" + phone.getNumber() + ")" +
                                    " hung up the phone. The call with phone_" + oldcPhone.getPhoneID() + " has been terminated.");
                            System.out.println(phone.getPhoneID() + "||" + (char) (phone.getIn_stationID() + 'a')
                                    + "--" + (char) (oldcPhone.getIn_stationID() + 'a') + "--" + oldcPhone.getPhoneID());
                            phone.setState(0);
                            oldcPhone.setState(0);
                        } else {
                            System.out.println("phone_" + phone.getPhoneID() + "(" + phone.getNumber() + ")" +
                                    " is still calling with phone_" + oldcPhone.getPhoneID() + "(" + oldcPhone.getNumber() + ")" +".");
                            System.out.println(phone.getPhoneID() + "--" + (char) (phone.getIn_stationID() + 'a')
                                    + "--" + (char) (oldcPhone.getIn_stationID() + 'a') + "--" + oldcPhone.getPhoneID());
                        }
                    }
                }
            }
            //处于空闲状态的话
            else {
                //如果在服务区
                if (phone.getIn_stationID() != -1) {
                    Phone cPhone = phones.get(cphone);
    //            System.out.println("!!!!!!!!");
    //            System.out.println(randomChoice());
    
                    if (randomChoice()) {
                        //50%的概率随机尝试拨打电话
                        System.out.println("phone_" + phone.getPhoneID() + "(" + phone.getNumber() + ")" +
                                "is trying to call phone_" + cphone);
                        //拨打对象不在服务区
                        if (cPhone.getIn_stationID() == -1) {
                            System.out.println("Failed, phone_" + cPhone.getPhoneID() + " is out of sevice!");
                            System.out.println(phone.getPhoneID() + "--" + (char) (phone.getIn_stationID() + 'a')
                                    + "||" + "NO BASESTATION" + "--" + cPhone.getPhoneID());
                            return;
                        }
                        //拨打对象处于通话中
                        if (cPhone.getState() == 1) {
                            System.out.println("Failed, phone_" + cPhone.getPhoneID() + " is on the line!");
                            System.out.println(phone.getPhoneID() + "--" + (char) (phone.getIn_stationID() + 'a')
                                    + "--" + (char) (cPhone.getIn_stationID() + 'a') + "||" + cPhone.getPhoneID());
                            return;
                        }
                        //否则通话成功
                        System.out.println("Call successful!");
                        System.out.println(phone.getPhoneID() + "--" + (char) (phone.getIn_stationID() + 'a')
                                + "--" + (char) (cPhone.getIn_stationID() + 'a') + "--" + cPhone.getPhoneID());
                        phone.setState(1);
                        phone.setCphoneID(cPhone.getPhoneID());
                        cPhone.setState(1);
                        cPhone.setCphoneID(phone.getPhoneID());
                    }
                    //50%的概率并不拨打电话
                    else{
                        System.out.println("phone_" + phone.getPhoneID() + "(" + phone.getNumber() + ")" +
                                "is not doing anything.");
                    }
                }
                //如果本机没在服务区，失败
                else{
                    System.out.println("phone_" + phone.getPhoneID() + "(" + phone.getNumber() + ")" +
                            " is out of service.The cell phone is currently unable to dial.");
                    System.out.println(phone.getPhoneID() + "||" + "NO BASESTATION");
    
                }
            }
    
        }
    
        //检查离手机最近的基站，返回id，失败返回-1
        public int checkStation(Phone phone) {
    
            int s;
            int min = 100000;
            int x = phone.getPosition_x();
            int y = phone.getPosition_y();
    
            for (BaseStation baseStation : baseStations) {
                s = (x - baseStation.getPosition_x()) * (x - baseStation.getPosition_x()) +
                        (y - baseStation.getPosition_y()) * (y - baseStation.getPosition_y());
                if (s <= min) min = s;
            }
            for (BaseStation baseStation : baseStations) {
                s = (x - baseStation.getPosition_x()) * (x - baseStation.getPosition_x()) +
                        (y - baseStation.getPosition_y()) * (y - baseStation.getPosition_y());
                if (s == min && s <= getR()) {
                    phone.setIn_stationID(baseStation.getStationID());
                    return phone.getIn_stationID();
                }
            }
            phone.setIn_stationID(-1);
            return -1;
    
        }
    
        //打印地图
        public void printMap(){
            int i, j;
            for (i = 0; i < width; i ++){
                for (j = 0; j < height; j ++){
                    System.out.print(getMap()[i][j]);
                    System.out.print(' ');
                }
                System.out.println();
            }
        }
    
        @Override
        public int randomNumber(int l, int r) {
            return (int)(Math.random() * (r  - l) + l);
        }
    
        @Override
        public boolean randomChoice() {
            double r = Math.random();
            return r < 0.5;
        }
    
    }

    ```

- **Main.java**
    ```JAVA
    import java.util.ArrayList;
    
    public class Main {
    
        public static void main(String[] args) {
            //参数设置
            int width = 15;     //地图宽度
            int height = 15;    //地图高度
            int R = 5;          //基站信号半径
            int baseStation_num = 26;   //基站数量
            int phone_num = 5;  //手机数量
            int times = 20;     //地图刷新（手机随机移动、随机拨号、随机挂断）
            ArrayList<Phone> phones = new ArrayList<>();
    
            int i;
            //生成手机
            for (i = 0; i < phone_num; i ++){
                Phone phone = new Phone(i, width, height);
                phones.add(phone);
            }
    
            //生成地图(含基站)
            ComMap comMap = new ComMap(width, height, R, baseStation_num, phone_num, phones);
    
            //刷新
            for (i = 0; i < times; i ++) comMap.refreshMap();
    
        }
    
    
    }

    ```
    
##4. 运行结果
![G-DMFR@DC3QK3XTUDYXURM](https://xusy-1300242514.cos.ap-nanjing.myqcloud.com/MWeb/20210606/gdmfrdc3qk3xtudyxurm.png)


![-w665](https://xusy-1300242514.cos.ap-nanjing.myqcloud.com/MWeb/20210606/sjd0pj8yrnjgxbewiak.png)

![123](https://xusy-1300242514.cos.ap-nanjing.myqcloud.com/MWeb/20210606/t5uhj5badw6yvb.png)

##5. 感想与体会
&emsp; 通过本次面向对象程序设计大作业，我更加深刻地理解了面向对象设计的含义。独自完成了各个类、类中的属性、方法以及各个接口的设计，在完成设计的过程中也遇到了很多问题，需要耐心地调试、切不可急躁，否则很难看出问题，因为出错的地方很有可能仅仅是几行代码乃至几个字母的错误。比如说，在 refreshMap() 时，发现各个手机始终都不进行操作，这个问题缠绕了很久，但经过仔细的查找，发现是 randomChoice() 函数返回值始终是 false，然后定位到了这个函数，发现return的值果然全是 false，这才解决了问题。
&emsp; 总而言之，这次大作业的设计让我亲自感受到了面向对象程序设计的内涵，让我对课程内容更加理解，丰富了我的动手实践经历，让我获取了更多实战的经验。
