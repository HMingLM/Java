[toc]
# GUI

### 1.概述
- Graphical User Interface(图形用户接口)
：用图形的方式来显示计算机操作的界面，更方便更直观；
- Java为GUI提供的对象都存在java.Awt(较依赖系统)和javax.Swing(移植性较强)两个包中；
- Container:容器，是一个特殊的组件，该组件中可以通过add方法添加其他组件进来；

### 2.布局
容器中组件的排放方式； 

- 常见的布局管理器

FlowLayout(流式)，BorderLayout(边界)，GirdLayout(网格)，CardLayout(卡片)，GirdBagLayout(网格包)；

### 3.Frame

创建图形化界面
- 创建frame窗体；
- 对窗体进行基本设置，如大小setSize()，出现的位置setLocation()，布局setLayout()；位置加大小setBounds(x,y,width,heigth);
- 定义组件,如按钮Button;
- 将组件通过窗体的add方法添加到窗体中；
- 让窗体显示，通过setVisible(true);

### 4.事件监听机制
组成
- **事件源(组件)**：就是awt包或者swing包中的那些图形界面组件；
- **事件(Event)**：每一个事件源都有自己特有的对应事件和共性事件；
- **监听器(Listener)**：将可以触发某一个事件的动作(不只一个)都已经封装到了监听器中；
- **事件处理(引发事件后的处理方式)**：对产生的动作进行处理；(我们要做的部分)

### 5.窗体事件
**创建监听器对象**，只要继承WindowAdapter并覆盖需要的方法即可；

因为WindowListener的子类WindowAdapter已经实现了WindowListener接口，并覆盖了其中所有方法，


```
import java.awt.*;
import java.awt.event.*;
class AwtDemo
{
    Frame f = new frame("my awt");
    f.setSize(500,400);
    f.setLocation(300,200);
    f.setLayout(new FlowLayout());
    Button b = new Button("我是一个按钮");
    f.add(b);
    
    f.addWindowListener(new MyWin());//创建监听器对象；
    f.setVisible(true);
}
class MyWin extends WindowAdapter
{
    public void WindowClosing(WindowEvent e)
    {
        //System.out.println(e.toString());
        System.exit(0); 
    }
}
```

创建监听器对象，调用关的方法，可以写成**匿名内部类**
- **windowClosing(WindowEvent e)**:当窗口被关闭时调用；
- **windowActivated(WindowEvent e)**：激活窗口时调用；
- **windowOpened(WindowEvent e)**：已打开窗口时调用；


```
f.addWindowListener(new WindowAdapter()
{
    public void windowClosing(WindowEvent e)
    {
        System.out.println("我关");
        System.exit(0);
    }
    public void windowActivated(WindowEvent e)
    {
        System.out.println("我激活了");
    }
    public void windowOpened(WindowEvent e)
    {
        System.out.println("我被打开了");
    }
});
```


### 6.Action事件

让按钮具备退出程序的功能
按钮就是事件源，想知道哪个组件具备什么样的特有监听器，要查看该组件对象的功能；

- **addActionListener(ActionListener l)**：添加指定的动作监听器，以接收发自此按钮的动作事件；
- 复写ActionListener中唯一的方法**actionPerformed(ActionEvent e)**;

```
but = new Button("my button");
but.addActionListener(new ActionListener()
{
    public void actionPerformed(ActionEvent e)
    {
        System.exit(0);
    }
}
```

### 7.鼠标事件

- **addMouseListener(MouseListener l)**:添加指定的鼠标监听器，以接收发自此组件的鼠标事件；(有适配器)

```
but.addMouseListener(new MouseAdapter()
{
    private int count = 1;
    private int clickCount = 1;
    public void mouseEntered(MouseEvent e)
    {
        System.out.println("鼠标进入到该组件"+count++);
    }
    public void mouseClicked(MouseEvent e)
    {
        System.out.println("点击动作"+clickCount++);
    }
});
```

**int getClickCount()**:返回与此事件关联的鼠标单击次数；

### 8.键盘事件

- **addKeyListener(KeyListener l)**：添加指定的按键监听器，以接收发自此组件的按键事件；(有适配器)


```
but.addKeyListener(new KeyAdapter()
{
    public void keyPressed(KeyEvent e)
    {
        if(e.isControlDown()&&e.getKeyCode()==KeyEvent.VK_ENTER)
        System.out.println("ctrl+enter is run");
        //System.out.println(KeyEvent.getKeyText(e.getKeyCode())+e.getKeyChar());
    });
}
```
##### 8.1 添加文本框组件
只可以输入数字；

```
private TextField tf;
tf = new TextField(20);//文本框；(列数)
f.add(tf);
tf.addKeyListener(new KeyAdapter()
{
    public void keyPressed(KeyEvent e)
    {
        int code = e.getKeyCode();
        if(!(code>=KeyEvent.VK_0 && code<=KeyEvent.VK_9))
        {
            System.out.println(code+"是非法的");
            e.consume();
        }
    }
});
```
- **TextField(int columns)**:文本框组件；
- **TextArea(int rows,int columns)**:构造新文本区，可以指定行数列数，并将空字符串作为文本；
- **consume()**：使用此事件，以便不会按照默认的方式由产生此事件的源代码来处理此事件；

##### 8.2 列出指定目录内容

```
private TextArea ta;
ta = new TextArea(25,70);
f.add(ta);
but.addActionListener(new ActionListener()
{
    public void actionPerformed(ActionEvent e)
    {
        showDir();
    }
});

private void showDir()
{
    String dirPath = tf.getText();
    File dir = new File(dirPath);
    if(dir.exists() && dir.isDirectory()) 
    {
        ta.setText("");
        String[] names = dir.list();
        for(String name : names)
        {
            ta.append(name+"\r\n");
        }
    }
    else
    {
        String info ="您输入的信息："+dirPath+"是错误的；请重输。";
        lab.setText(info);
        d.setVisible(true);
    }
}
```

### 9.对话框Dialog
Dialog(Frame owner，String title,boolean modal):modal为true时，不操作对话框则无法操作窗体；为flase时相反；

```
private Dialog d;
private Label lab;
private Button okBut;

d = new Dialog(f,"提示信息-self",true);
d.setBounds(400,200,240,150);
d.setLayout(new FolwLayout());

lab = new Label();
okBut = new Button("确定");

d.add(lab);
d.add(okBut);

else//放的位置如8.2
{
    String info = "您输入的信息："+dirPath+"是错误的；请重输。";
    lab.setText(info);
    d.setVisible(true);
}

d.addWindowListener(new WindowAdapter()
{
    public void windowClosing(WindowEvent e)
    {
        d.setVisible(flase);
    }
});

okBut.addActionListener(new ActionListener()
{
    public void actionPerformed(ActionEvent e)
    {
        d.setVisible(flase);
    }
});

tf.addKeyListener(new KeyAdapter()
{
    public void keyPressed(KeyEvent e)
    {
        if(e.getKeyCode()==KeyEvent.VK_ENTER)
            showDir();
    }
});
```
### 9.菜单

```
private Frame f;
private MenuBar mb;//菜单栏；
private Menu m,subMenu;//菜单;
private MenuItem closeItem,subItem;//子条目；

f = new Frame("my window");
f.setBounds(300,100,500,600);
f.setLayout(new FlowLayout());

mb = new MenuBar();

m = new Menu("文件")；
subMenu = new Menu("子菜单");

subItem = new MenuItem("子条目");
closeItem = new MenuItem("退出");

f.setMenuBar(mb);
mb.add(m);
m.add(subMenu);
m.add(closeItem);
subMenu.add(subItem);

myEvent();
f.setVisible(true);

private void myEvent()
{
    closeItem.addActionListener(new ActionListener()
    {
        public void actionPerformed(ActionEvent e)
        {
            System.exit(0);
        }
    });
    f.addWindowListener(new WindowAdapter()
    {
        public void windowClosing(WindowEvent e)
        {
            System.exit(0);
        }
    });
}
```
### 10.打开文件，保存文件
- **FileDialog(Frame parent,String title,int mode)**:创建一个有指定标题的文件对话框，用于加载或保存文件；

```
private MenuItem openItem,saveItem,closeItem;

private TextArea ta;
ta = new TextArea();
//再去掉布局设置；
private File file;
private FielDialog openDia,saveDia;
openDia = new FileDialog(f,"我要打开",FileDialog.LOAD);
saveDia = new FileDialog(f,"我要保存",FileDialog.SAVE);

private void myEvent()
{
    openItem.addActionListener(new ActionListener()
    {
        public void actionPerformed(ActionEvent e)
        {
            openDia.setVisible(true);
            String dirPath = openDia.getDirectory();
            String fileName = openDia.getFile();
            if(dirPath==null || fileName==null)
                return ;
            ta.setText("");
            file = new File(dirPath,fileName);
            try
            {
                BufferedReader bufr = new BufferedReader(new FileReader(file));
                String line = null;
                while((line=bufr.readLine())!=null)
                {
                    ta.append(line+"\r\n");
                }
                bufr.close();
            }
            catch(IOException ex)
            {
                throw new RuntimeException("读取失败")；
            }
        }
    });
    saveItem.addActionListener(new ActionListener()
    {
        public void actionPerformed(ActionEvent e)
        {
            if(file==null)
            {
                saveDia.setVisible(true);
                String dirPath = saveDia.getDirectory();
                String fileName = saveDia.getFile();
                if(dirPath==null || fileName==null)
                    return ;
                file = new File(dirPath,fileName);
            }
            try
            {
                BufferedWriter bufw = new BufferedWriter(new FileWriter(file));
                String text = ta.getText();
                bufw.write(text);
                bufw.close();
            }
            catzh(IOException ex)
            {
                throw new RuntimeException();
            }
        }
    });
}
```
最后打包成jar，能够双击运行；