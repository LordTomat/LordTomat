using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Runtime.InteropServices;
using System.Threading;
namespace Belarus_Bellum
{

    public partial class Gameplay : Form
    {
        int Eid; //Запоминает айди ивента (для последствий)
        String Text; //Используется для передачи новостей
        bool AutoBattle = true; //Происходят ли битвы в автоматическом режиме (без тактик)
        int CityBot; //Используется для запоминания айдишника города, ИЗ которого атакует бот. Игрок использует айдишник City в зависимости от открытого меню и его лучше не трогать
        int Day = 3;
        int Month = 4; //Это стоит объяснять?
        int Year = 2021;
        int leadAtk; //Айдишник того, кто атакует поселение. Если он не совпадает с lead = то атакует не игрок.
        int MapSpeed = 100; //Скорость прокрутки карты + глобальная задача координат поселениям 
        short[] CordX = {      1740,     1500,      325,      240,    2600,     2360,      2300,     1865,      2025,     2380,        990,      1005,     777,    2045,     2285,      1480,      1280,         1537,          1620,        444,     1140,}; //X координата на карте
        short[] CordY = {      1120,     1215,     1300,     2150,    2000,     1200,       500,      345,      1630,      870,       1630,      2180,    1200,    2205,     1762,      1825,       985,          538,           190,       2100,      900,}; //Y координата
        String[] CityName = { "Жодино", "Минск", "Гродно", "Брест", "Гомель", "Могилёв", "Витебск", "Полоцк", "Бобруйск", "Орша", "Барановичи", "Пинск", "Лида", "Мозырь", "Жлобин", "Солигорск", "Молодечно", "Глубокое", "Верхнедвинск", "Кобрин", "Сморгонь" }; //Название городов. Номер массива = айди города                                                                                                                                                                                                          
        //                        0         1         2        3         4       5            6         7          8         9          10         11      12       13         14         15         16            17            18           19         20
        int[] CityOwner = {       0,        0,        1,        3,       4,        9,         5,        6,        12,        2,         10,         7,       8,      11,       12,        12,        12,           12,           12,          13,        14,}; //Кто владеет городом с айди массива. Номер = айди владельца
        int[] CFactory = {        2,       10,        7,        6,       8,        7,         6,        4,         3,        1,          3,         4,       2,       6,        4,         8,         4,            1,            0,           3,         1, }; // Количество заводов в городах. Массив = айди города. С - сокращение от City
        int[] CFactoryWeapon = {  1,        8,        2,        4,       5,        3,         2,        1,         1,        3,          1,         3,       0,       5,        1,         0,         2,            0,            0,           0,         0,}; //Оружейные заводы
        int[] CFactoryTank = {    5,       10,        2,        4,       6,        1,         3,        0,         1,        3,          1,         1,       0,       4,        5,         0,         0,            0,            1,           0,         1,}; //Танковые
        int[] People = {      65000,  1975000,   368000,   339700,  508839,   380440,    362466,    84597,    217940,   115938,     179439,    138202,  101165,  111801,    76078,    106839,    101300,        18921,         7335,       52843,     36283,}; //Кол. людей живущих в городе на старте игры. Используется для подсчёта процента призыва. Айди массива = айди города
        int[] CPath1 = {          1,        0,        1,        2,      11,        1,         7,        6,         1,        6,         19,        15,       2,       4,       13,         8,         1,           16,            7,           3,        16,};  //Путь от города. Максимум 5 развилок от 1 города
        int[] CPath2 = {          9,        8,        0,       11,       5,        0,         5,       17,         5,        5,          1,        19,       7,       8,        8,        11,        12,            7,           17,          10,        17,};
        int[] CPath3 = {          5,        5,       10,        8,       8,        9,         9,       18,        15,        0,          8,         4,      16,      11,        4,        -1,        17,           18,           -1,          11,        12,};
        int[] CPath4 = {          2,        9,       12,       19,      13,        8,         0,       -1,        13,       -1,         -1,        13,      20,      14,       -1,        -1,        20,           20,           -1,          -1,        -1,};
        int[] CPath5 = {         -1,       16,       -1,       -1,      14,       -1,        -1,       -1,        14,       -1,         -1,        -1,      -1,      -1,       -1,        -1,        -1,           -1,           -1,          -1,        -1,};
        int[] Fort = {            0,        3,        1,        3,       1,        1,         1,        0,         0,        1,          2,         0,       0,       1,        0,         1,         0,            0,            0,           0,         1,}; //Защитные форты в городе
        int[] GarPeasant = {      0,        0,       50,      100,      10,        0,        30,        0,         0,      100,          0,        50,     200,       0,        0,        80,        30,            0,            5,         100,         0,}; //Сколько в городе крестьян
        int[] GarSolder = {      20,      100,       20,       20,      30,      100,        50,       20,        20,        0,         80,         0,       0,      20,       50,         0,        10,           20,            0,           0,        50,}; //Сколько в городе солдат
        int[] GarTank = {         3,       10,        0,       10,      30,       10,         5,        1,         0,       10,          6,        30,       0,       3,        0,         0,         0,            0,            1,           0,        12,}; //Сколько в городе танков

        int[] PeopleLive = new int[200]; //Тоже, что и выше, но сколько людей в городе осталось
        String[] NameLead = { "Александр Лукашенко", "СНО", "Автономная Орша", "Алексей Сокол", "Сергей Чижик", "Андрей Равков", "Евгений Васькович", "Анархисты", "ОЛСБ", "Владимир Цумарев", "Виктор Дичковский", "Юрий Караев", "Отряды карателей", "Павел Северинец",  "Еврейское собрание" }; //Имена с айди владельцами
        //                               0             1           2                3                4                5                 6                 7           8             9                 10                11                 12                  13                   14
        int[] Stab = {                  30,           50,         50,              50,              50,              60,               40,               20,         90,           10,                20,               50,                 3,                 80,                  90,}; //Аналог стабильности из хойки (или народного единства, кому как)
        double[] CanRecrut = {         0.5,         0.25,       0.25,            0.25,            0.25,             0.4,             0.25,                1,       0.25,          0.1,               0.2,              0.5,               0.1,                0.3,                 0.2, }; // Процент призывного населения
        double[] Money = {             100,          100,        100,             100,             100,             200,              100,              100,        100,          100,               100,               50,                 0,                140,                 200, }; //Стартовые деньги. Айди массива = айди владельца
        double[] Weapon = {            100,          100,        100,             100,             100,              25,              100,              100,        100,          100,               100,              150,                 1,                 20,                  30, }; //Кол. оружия у айди владельца (для найма)
        double[] Tank = {              100,          100,        100,             100,             100,             100,              100,              100,        100,          100,               100,               50,                 0,                  0,                  40, }; //Кол. танков у айди владельца (для найма)
        double[] PricePeasantGold = {    1,            1,          1,               1,               1,               1,                1,                1,          1,            1,                 1,                1,                 3,                  1,                   2,}; //Сколько стоит 1 крестьянин для каждого отдельного лидера
        double[] PriceSolderGold = {     1,            1,          1,               1,               1,               1,                1,                1,          1,            1,                 1,                1,                 5,                  1,                   1,}; //Сколько стоит 1 солдат для каждого отдельного лидера
        double[] PriceSolderWeapon = {   1,            1,          1,               1,               1,               1,                1,                1,          1,            1,                 1,                1,                 1,                  1,                   1,}; //Сколько стоит 1 солдат в оружии для каждого отдельного лидера
        double[] PriceTankGold = {       2,            2,          2,               2,               2,               2,                2,                1,          2,            2,                 2,                1,                 7,                  3,                   1,}; //Сколько стоит 1 танк для каждого отдельного лидера
        double[] PriceTankTank = {       1,            1,          1,               1,               1,               1,                1,                1,          1,            1,                 1,                1,                 1,                  1,                   1,}; //Сколько стоит 1 танк в танках (не смешно) для каждого отдельно лидера
        Byte[] Argb1 = {               255,            0,         15,             200,               0,               0,              255,              100,         10,          200,               200,               20,                 1,                 10,                   0,}; //Крансный
        Byte[] Argb2 = {                 0,            0,          0,              50,             100,             255,              255,                5,         10,          200,               200,              140,                 1,                120,                 255, }; //Зеленый
        Byte[] Argb3 = {                 0,          255,          0,              50,             255,               0,              255,               20,         40,          255,               255,              140,                 1,                255,                 255,}; //Синий
        Bitmap[] LeadPortret = { Properties.Resources.Lyka, Properties.Resources.SNO, Properties.Resources.Orsha, Properties.Resources.Sokol, Properties.Resources.Chijik, Properties.Resources.Ravkov, Properties.Resources.Vasya, Properties.Resources.Anarhy, Properties.Resources._21615, Properties.Resources._21615, Properties.Resources._21615, Properties.Resources.karaev, Properties.Resources.OMON, Properties.Resources.Sever, Properties.Resources.Jew, };
        //Множители снизу создал в виде массива (и не целого числа) потому, что в случае чего планирую "влиять" на них путём всяких фокусов и прочих приколов. Отправлять в бой половину рекрута никто не будет, а вот с 1 винтовкой на троих - вполне.

        int City; //Запоминает айди последнего "тыкнутого" города. На него отсылается найм, например
        int Xrad1; //4 переменные используются для более удобного подсчёта того, куда тыкает игрок, но по хорошеу их лучше заменить
        int Xrad2;
        int Yrad1;
        int Yrad2;
        bool RealTimeOn = true; //  Пауза/реальное время
        public static int lead; // Айди лидера, переносится из первой формы
        public static bool MusicON; //  Используется для вкл/выкл музыки из первой формы
        Random rnd = new Random(); //рандом
        WMPLib.WindowsMediaPlayer Player; //для музыки
        byte t = 0; //Помогает отсчитывать инком
        int CityTarget = 0; // Используется при запоминании, в какой город двигаются войска
        int[] AtkNum = { 0, 0, 0, 0, 0, 0, 0 }; //Запоминает число атакующих войск
        int D; //Я уже забыл где это юзалось, но оно юзалось, поэтому не трогаю...
        bool[] CanFocus = new bool[30]; //Проверяет, должна ли показываться кнопка фокуса
        PictureBox[] TestFocus = new PictureBox[18]; //Используется для запоминания фокуса в массив, потом в совокупности с верхней переменной проверяет, должен ли ЭТОТ фокус показываться
        string[] FocusInfo = new string[20]; //Запоминает текст фокуса (название, описание, бонусы. Весь сразу!) и показывает его при наведении мышкой
        bool[] DestroyLimit = new bool[200]; //Позволяет ограничивать 1 разрушение в городе в день
        int[] DefNum = new int[4]; //Запоминает кол. защитников
        byte Testing = 0; //Для теста новой системы передвижения камеры
        public Gameplay()
        {
            InitializeComponent();
            Close_Menu();
            EventShutdown();
            Sec10.Stop();
            Update_Point();
            MenuPanel.Visible = false;
            FocusPanel.Visible = false;
            RealTimeOn = false;
            EventPanel.Visible = false;
            TimerPicture.BackgroundImage = Properties.Resources.TimeStop;
            VictoryScreen.Visible = false;
            Console.WriteLine("Инициализировалось");
            this.Location = this.PointToScreen(new Point(0, 0));
            for (int i = 0; i < People.Length; i++)
            {
                PeopleLive[i] = People[i];
                D = i;
                CanFocus[i] = false;
            }
            Update_Text();
            NewsPanel.Size = new Size(470, 160);
            NewsPanel.Location = new Point(535, 560);
    }

        protected override bool ProcessCmdKey(ref Message msg, Keys keyData)
        {
            switch (keyData)
            {
                case Keys.Left:
                    if (BelarusMap.Left <= 0 && BelarusMap.Left >= -2200)
                    {
                        BelarusMap.Left += MapSpeed;
                        if (BelarusMap.Left < -2200)
                        {
                            BelarusMap.Left = -2200;
                        }
                        if (BelarusMap.Left > 0)
                        {
                            BelarusMap.Left = 0;
                        }
                    }
                    break;
                case Keys.Right:
                    if (BelarusMap.Left <= 0 && BelarusMap.Left >= -2200)
                    {
                        BelarusMap.Left -= MapSpeed;
                        if (BelarusMap.Left < -2200)
                        {
                            BelarusMap.Left = -2200;
                        }
                        if (BelarusMap.Left > 0)
                        {
                            BelarusMap.Left = 0;
                        }
                    }
                    break;
                case Keys.Up:
                    if (BelarusMap.Top <= 0 && BelarusMap.Top >= -1900)
                    {
                        BelarusMap.Top += MapSpeed;
                        if (BelarusMap.Top < -1900)
                        {
                            BelarusMap.Top = -1900;

                        }
                        if (BelarusMap.Top > 0)
                        {
                            BelarusMap.Top = 0;
                        }
                    }
                    break;
                case Keys.Down:
                    if (BelarusMap.Top <= 0 && BelarusMap.Top >= -1900)
                    {
                        BelarusMap.Top -= MapSpeed;
                        if (BelarusMap.Top < -1900)
                        {
                            BelarusMap.Top = -1900;
                        }
                        if (BelarusMap.Top > 0)
                        {
                            BelarusMap.Top = 0;
                        }
                    }
                    break;
                case Keys.Space:
                    if (TimerPicture.Visible == true)
                    {
                        if (RealTimeOn == true)
                        {
                            Sec10.Stop();
                            RealTimeOn = false;
                            TimerPicture.BackgroundImage = Properties.Resources.TimeStop;
                        }
                        else
                        {
                            Sec10.Start();
                            RealTimeOn = true;
                            TimerPicture.BackgroundImage = Properties.Resources.TimePlay;
                        }
                    }
                    break;
            }
            return base.ProcessCmdKey(ref msg, keyData);
        } //Обновлённая система движения
        private void BelarusMap_Click(object sender, EventArgs e)
        {
            Close_Menu();
            Xrad1 = Cursor.Position.X - BelarusMap.Left + 40;
            Xrad2 = Cursor.Position.X - BelarusMap.Left - 40;
            Yrad1 = Cursor.Position.Y - BelarusMap.Top + 40;
            Yrad2 = Cursor.Position.Y - BelarusMap.Top - 40;
            //MessageBox.Show(string.Format("X: {0} Y: {1}.", Cursor.Position.X - BelarusMap.Left, Cursor.Position.Y - BelarusMap.Top));
            for (int i = 0; i < CityName.Length; i++)
            {
                if (CordY[i] <= Yrad1 && CordY[i] >= Yrad2 && CordX[i] <= Xrad1 && CordX[i] >= Xrad2)
                {
                    CityMenuBack.Visible = true;
                    LaberCity.Visible = true;
                    if (CityOwner[i] == lead)
                    {
                        WhoControlTown.Text = "Это ваше поселение";
                        CityArmyManager.Visible = true;
                        CityPromManager.Visible = true;
                        CityCharge.Visible = true;
                        CityDestroy.Visible = true;
                    }
                    else
                    {
                        WhoControlTown.Text = "Этот город контролирует " + NameLead[CityOwner[i]];
                    }
                    WhoControlTown.Visible = true;
                    MenuClose.Visible = true;
                    MenuClose.BringToFront();
                    City = i;
                    Update_Text();
                    i = 999;
                }
                else
                {
                    City = -1;
                }
            }
        }
        private void MusicTimer_Tick(object sender, EventArgs e)
        {
            if (MusicON == true)
            {
                Player = new WMPLib.WindowsMediaPlayer();
                int i = rnd.Next(1, 6); // Всегда делать рандом на 1 меньше, чем максимальный. Т.е. если rnd.Next(1, 9), то последний if = 8
                if (i == 1)
                {
                    MusicTimer.Interval = 262000; //1000 = 1 секунда
                    Player.URL = @"https://oxy.sunproxy.net/file/MVFKZ1dNd2hnNldaZ21mNitqUHVoTG9zSEpIVEd3eXpTcXZ6dkk1YlQ3MVFXR3c0cDFWSzFrQXlteWlzZVcya1doYWxZNjhNMTYzbExCVjROcVArY1ZyVTlYUWFpbzRHZ3Y1VkgyeUJYaGs9/TOR_BAND_-_Rodnaya_zyamlya_(Vuxo.org).mp3";
                }
                if (i == 2)
                {
                    MusicTimer.Interval = 204000;
                    Player.URL = @"https://muzish.net/uploads/music/2020/09/Lavon_Volski_Voragi_narodu.mp3";
                }
                if (i == 3)
                {
                    MusicTimer.Interval = 271000;
                    Player.URL = @"https://mn1.sunproxy.net/file/WDhRNnBocGlLYkdyWWNJc3lXTjdGZnhwbnh5ZHM2QkNsOWVVZ2d4K0J2MnM2dUdUQmRrQmdtSktyOFhQZmQwd2tHeXExSVFLNEUyVmtNOVNkdVdCRFZrZVlRZ1J5TFJ3UHBiSzhwQytiR3c9/symbal.by_-_Razbury_turmy_mury_._Spyavayuc_Syargej_Kosmas_Syargej_C_hano_sk_(mp3.mn).mp3";
                }
                if (i == 4)
                {
                    MusicTimer.Interval = 259000;
                    Player.URL = @"https://mn1.sunproxy.net/file/WDhRNnBocGlLYkdyWWNJc3lXTjdGZnhwbnh5ZHM2QkNsOWVVZ2d4K0J2MWY5d3RkNWFrZWpob1BVU09ST2t0aFZEeXRzbkdReGhWeE8zcUhuckdYZW51NnRJdlFiMFY5VEdJaWpwZDNSTnM9/Lavon_Volski_-_Lohkija-lohkija_(mp3.mn).mp3";
                }
                if (i == 5)
                {
                    MusicTimer.Interval = 231000;
                    Player.URL = @"https://mn1.sunproxy.net/file/WDhRNnBocGlLYkdyWWNJc3lXTjdGZnhwbnh5ZHM2QkNsOWVVZ2d4K0J2MFI3TXhiRFpuM0pQUjVSNWhVSUF6U3IxZkQ4b2tiTmhVYmZtNThSdnl1T3AyeTBqbHBFSy9TQWJUZmU2WDFQbEk9/Bely_Leg_yon_-_Voiny_ranku_(mp3.mn).mp3";
                }
                Player.controls.play();
            }
            else
            {
                MusicTimer.Stop();
            }
        }

        private void Sec10_Tick(object sender, EventArgs e)
        {
            if (t == 5)
            {
                t = 0;
                Day++;
                if (Day == 5 || Day == 10 || Day == 15 || Day == 20 || Day == 25 || Day == 30)
                {
                    for (int i = 0; i < CityOwner.Length; i++)
                    {
                        People[i] = PeopleLive[i];
                    }
                }
                if (Day == 31)
                {
                    Day = 1;
                    Month++;
                }
                if (Month == 13)
                {
                    Month = 1;
                    Year++;
                }
                GameTime.Text = "Полночь, " + Day + "." + Month + "." + Year;
                for (int i = 0; i < CityOwner.Length; i++)
                {
                    Money[CityOwner[i]] += Convert.ToInt32(CFactory[i] * 50 * Stab[CityOwner[i]] * 0.01);
                    Weapon[CityOwner[i]] += Convert.ToInt32(CFactoryWeapon[i] * 10 * Stab[CityOwner[i]] * 0.01);
                    Tank[CityOwner[i]] += Convert.ToInt32(CFactoryTank[i] * 10 * Stab[CityOwner[i]] * 0.01);
                    Victory();
                }
                if (lead == 0 && Day == 4 && Month == 4 && Year == 2021)
                {
                    Eid = 1;
                    EventLoad();
                }
                if (lead == 0 && Day == 30 && Month == 8)
                {
                    Eid = 8;
                    EventLoad();
                }
                if (lead == 1 && Day == 4 && Month == 4 && Year == 2021)
                {
                    Eid = 4;
                    EventLoad();
                }
                if (lead == 2 && Day == 4 && Month == 4 && Year == 2021)
                {
                    Eid = 5;
                    EventLoad();
                }
                if (lead == 3 && Day == 4 && Month == 4 && Year == 2021)
                {
                    Eid = 7;
                    EventLoad();
                }
                if (lead == 4 && Day == 4 && Month == 4 && Year == 2021)
                {
                    Eid = 9;
                    EventLoad();
                }
                if (lead == 8 && Day == 4 && Month == 4 && Year == 2021)
                {
                    Eid = 10;
                    EventLoad();
                }
                if (lead == 11 && Day == 4 && Month == 4 && Year == 2021)
                {
                    Eid = 11;
                    EventLoad();
                }
                for (int i = 0; i < CityOwner.Length; i++)
                {
                    DestroyLimit[i] = false;
                }
            }
            else
            {
                if (lead == 7 && Day == 3 && Month == 4 && Year == 2021 && t == 2)
                {
                    Eid = 2;
                    EventLoad();
                }
                if (lead == 5 && Day == 3 && Month == 4 && Year == 2021 && t == 1)
                {
                    Eid = 3;
                    EventLoad();
                }
                if (lead == 6 && Day == 3 && Month == 4 && Year == 2021 && t == 0)
                {
                    Eid = 6;
                    EventLoad();
                }
                if (lead == 14 && Day == 3 && Month == 4 && Year == 2021 && t == 1)
                {
                    Eid = 13;
                    EventLoad();
                }
                switch (t)
                {
                    case 0:
                        GameTime.Text = "Ночь, " + Day + "." + Month + "." + Year;
                        break;
                    case 1:
                        GameTime.Text = "Утро " + Day + "." + Month + "." + Year;
                        break;
                    case 2:
                        GameTime.Text = "Полдень " + Day + "." + Month + "." + Year;
                        break;
                    case 3:
                        GameTime.Text = "Ранний вечер " + Day + "." + Month + "." + Year;
                        break;
                    case 4:
                        GameTime.Text = "Глубокий вечер " + Day + "." + Month + "." + Year;
                        break;
                }
                t++;
            }
            for (int i = 0; i < CityOwner.Length; i++)
            {
                if (CityOwner[i] != lead) //Если городом владеет не игрок
                {
                    System.Console.WriteLine("Город " + CityName[i] + " не принадлежит игроку, бот с айди " + CityOwner[i] + " думоет! А ещё это " + i + " операция найма");
                    if (GarPeasant[i] == 0 && GarSolder[i] == 0 && GarTank[i] == 0) //И там нету гарнизона от слова совсем
                    {
                        System.Console.WriteLine("В этом городе обнаружили пустой гарнизон! Кошмар какой...");
                        int C = (int)(((People[i] * CanRecrut[CityOwner[i]]) / 100) - (People[i] - PeopleLive[i])); //Мы высчитываем количество рекрутов 
                        System.Console.WriteLine("Наскребли " + C + " призывников...");
                        if (C >= 1 && Money[CityOwner[i]] >= PricePeasantGold[CityOwner[i]]) //И проверяем, есть ли у нас вообще на что либо деньги
                        {
                            System.Console.WriteLine("Бабло тоже есть");
                            if (Weapon[CityOwner[i]] >= PriceSolderWeapon[CityOwner[i]] && Money[CityOwner[i]] >= PriceSolderGold[CityOwner[i]]) //А если есть ещё и оружие, хоть 1 штука...
                            {
                                System.Console.WriteLine("Нашли оружия");
                                int M = (int)(Money[CityOwner[i]] / PriceSolderGold[CityOwner[i]]); //На сколько солдатиков нам хватает денег
                                int W = (int)(Weapon[CityOwner[i]] / PriceSolderWeapon[CityOwner[i]]); //И оружия
                                int R = M; // Для начала устанавливаем, что можем нанять солдатиков = сколько хватает денег
                                if (M > W) { R = W; } //Теперь сверяем, а хватит ли на всех стволов
                                if (W > C) { R = C; } //И хватит ли вообще людей
                                PeopleLive[i] -= R;
                                GarSolder[i] += R;
                                Money[CityOwner[i]] -= (PriceSolderGold[CityOwner[i]] * R);
                                Weapon[CityOwner[i]] -= (PriceSolderWeapon[CityOwner[i]] * R);
                                C -= R;
                                System.Console.WriteLine("И завербовали " + R + " солдат");
                            }
                            if (Tank[CityOwner[i]] >= PriceTankTank[CityOwner[i]] && Money[CityOwner[i]] >= PriceTankGold[CityOwner[i]]) //На складе есть танки!
                            {
                                System.Console.WriteLine("Нашли танки");
                                int M = (int)(Money[CityOwner[i]] / PriceTankGold[CityOwner[i]]);
                                int W = (int)(Tank[CityOwner[i]] / PriceTankTank[CityOwner[i]]);
                                int R = M;
                                if (M > W) { R = W; }
                                if (W > C) { R = C; }
                                PeopleLive[i] -= R;
                                GarTank[i] += R;
                                Money[CityOwner[i]] -= (PriceTankGold[CityOwner[i]] * R);
                                Tank[CityOwner[i]] -= (PriceTankTank[CityOwner[i]] * R);
                                C -= R;
                                System.Console.WriteLine("И завербовали " + R + " танкистов");
                            }
                            if (Money[CityOwner[i]] >= PricePeasantGold[CityOwner[i]]) //На складе есть танки!
                            {
                                System.Console.WriteLine("Либо хватило рекрутов, либо не хватило винтовок с танками, но решили нанять ополченцев");
                                int M = (int)(Money[CityOwner[i]] / PricePeasantGold[CityOwner[i]]);
                                int R = M;
                                if (M > C) { R = C; }
                                PeopleLive[i] -= R;
                                GarPeasant[i] += R;
                                Money[CityOwner[i]] -= (PricePeasantGold[CityOwner[i]] * R);
                                C -= R;
                                System.Console.WriteLine("И завербовали " + R + " крестьян, помолимся...");
                            }
                        }
                    }
                    else
                    {
                        int R = rnd.Next(1, 6);
                        CityBot = i;
                        if (CPath1[i] != -1 && CityOwner[CPath1[i]] != CityOwner[i] && Day != 3 && R == 1)
                        {
                            CityTarget = CPath1[i];
                            BotTryMoveToCity();
                            goto EndExp;
                        }
                        R = rnd.Next(1, 6);
                        if (CPath2[i] != -1 && CityOwner[CPath2[i]] != CityOwner[i] && Day != 3 && R == 1)
                        {
                            CityTarget = CPath2[i];
                            BotTryMoveToCity();
                            goto EndExp;
                        }
                        R = rnd.Next(1, 6);
                        if (CPath3[i] != -1 && CityOwner[CPath3[i]] != CityOwner[i] && Day != 3 && R == 1)
                        {
                            CityTarget = CPath3[i];
                            BotTryMoveToCity();
                            goto EndExp;
                        }
                        R = rnd.Next(1, 6);
                        if (CPath4[i] != -1 && CityOwner[CPath4[i]] != CityOwner[i] && Day != 3 && R == 1)
                        {
                            CityTarget = CPath4[i];
                            BotTryMoveToCity();
                            goto EndExp;
                        }
                        R = rnd.Next(1, 6);
                        if (CPath5[i] != -1 && CityOwner[CPath5[i]] != CityOwner[i] && Day != 3 && R == 1)
                        {
                            CityTarget = CPath5[i];
                            BotTryMoveToCity();
                            goto EndExp;
                        }
                    EndExp:
                        System.Console.WriteLine("пук");
                    }
                }
            }
            Update_Text();
            System.Console.WriteLine(t);
        }

        public void Update_Point()
        {
            Bitmap bmp = new Bitmap(BelarusMap.Width, BelarusMap.Height);
            Graphics g = Graphics.FromImage(bmp);
            Pen RPen = new Pen(Color.Red, 10);
            Pen BPen = new Pen(Color.Blue, 10);
            for (int i = 0; i < CityOwner.Length; i++)
            {
                Color Testt = Color.FromArgb(Argb1[CityOwner[i]], Argb2[CityOwner[i]], Argb3[CityOwner[i]]);
                Pen Test = new Pen(Testt,10);
                g.DrawEllipse(Test, CordX[i] - 20, CordY[i] - 20, 40, 40);
            }
            BelarusMap.Image = bmp;
        }

        public void Close_Menu() // Должен срабатывать всегда, когда предполагается закрыть меню/поменять окно.
        {
            CityPromManager.Visible = false;
            CityMenuBack.Visible = false;
            CityArmyManager.Visible = false;
            HirePeasant.Visible = false;
            TrackBarPeasant.Visible = false;
            HowManyPeasant.Visible = false;
            HireSolder.Visible = false;
            TrackBarSolder.Visible = false;
            HowManySolder.Visible = false;
            HireTank.Visible = false;
            TrackBarTank.Visible = false;
            HowManyTank.Visible = false;
            ArmyStatusCity.Visible = false;
            BuildFactory.Visible = false;
            BuildFactoryTank.Visible = false;
            BuildFactoryWeapon.Visible = false;
            HowFactory.Visible = false;
            HowFactoryTank.Visible = false;
            HowFactoryWeapon.Visible = false;
            CityCharge.Visible = false;
            Charge1.Visible = false;
            Charge2.Visible = false;
            Charge3.Visible = false;
            Charge4.Visible = false;
            Charge5.Visible = false;
            CityCharge.Visible = false;
            HowChargePeasant.Visible = false;
            HowChargeSolder.Visible = false;
            HowChargeTank.Visible = false;
            TrackChargePeasant.Visible = false;
            TrackChargeSolder.Visible = false;
            TrackChargeTank.Visible = false;
            MenuBack.Visible = false;
            CityDestroy.Visible = false;
            DestroyYesUCan.Visible = false;
            DestroyFactoryText.Visible = false;
            DestroyFortText.Visible = false;
            DestroyWeaponText.Visible = false;
            DestroyTankText.Visible = false;
            DestroyPeopleText.Visible = false;
            DestroyPeopleButt.Visible = false;
            DestroyTankButt.Visible = false;
            DestroyWeaponButt.Visible = false;
            DestroyFactoryButt.Visible = false;
            DestroyFortButt.Visible = false;
        }

        private void CityArmyManager_Click(object sender, EventArgs e)
        {
            Close_Menu();
            CityMenuBack.Visible = true;
            LiveShow.Visible = true;
            WhoControlTown.Visible = true;
            LaberCity.Visible = true;
            TrackBarPeasant.Visible = true;
            HowManyPeasant.Visible = true;
            HirePeasant.Visible = true;
            TrackBarSolder.Visible = true;
            HowManySolder.Visible = true;
            HireSolder.Visible = true;
            TrackBarTank.Visible = true;
            HowManyTank.Visible = true;
            HireTank.Visible = true;
            ArmyStatusCity.Visible = true;
            MenuBack.Visible = true;
            ArmyTrack();
            Update_Text();
        }

        private void TrackBarPeasant_Scroll(object sender, EventArgs e)
        {
            Update_Text();
        }

        public void ArmyTrack()
        {
            TrackBarPeasant.Maximum = (int)(((People[City] * CanRecrut[lead]) / 100) - (People[City] - PeopleLive[City]));
            TrackBarSolder.Maximum = (int)(((People[City] * CanRecrut[lead]) / 100) - (People[City] - PeopleLive[City]));
            TrackBarTank.Maximum = (int)(((People[City] * CanRecrut[lead]) / 100) - (People[City] - PeopleLive[City]));
            TrackBarPeasant.Minimum = 0;
            TrackBarSolder.Minimum = 0;
            TrackBarTank.Minimum = 0;
            if (Money[lead] / PricePeasantGold[lead] < TrackBarPeasant.Maximum)
            {
                TrackBarPeasant.Maximum = (int)(Money[lead] / PricePeasantGold[lead]);
            }
            if (Money[lead] / PriceSolderGold[lead] < TrackBarSolder.Maximum)
            {
                TrackBarSolder.Maximum = (int)(Money[lead] / PriceSolderGold[lead]);
            }
            if (Weapon[lead] / PriceSolderWeapon[lead] < TrackBarSolder.Maximum)
            {
                TrackBarSolder.Maximum = (int)(Weapon[lead] / PriceSolderWeapon[lead]);
            }
            if (Money[lead] / PriceTankGold[lead] < TrackBarTank.Maximum)
            {
                TrackBarTank.Maximum = (int)(Money[lead] / PriceTankGold[lead]);
            }
            if (Tank[lead] / PriceTankTank[lead] < TrackBarTank.Maximum)
            {
                TrackBarTank.Maximum = (int)(Tank[lead] / PriceTankTank[lead]);
            }
        }

        private void HirePeasant_Click(object sender, EventArgs e)
        {
            if (TrackBarPeasant.Value > 0)
            {
                PeopleLive[City] -= TrackBarPeasant.Value;
                GarPeasant[City] += TrackBarPeasant.Value;
                Money[lead] -= (PricePeasantGold[lead] * TrackBarPeasant.Value);
                TrackBarPeasant.Maximum -= TrackBarPeasant.Value;
                TrackBarPeasant.Value = 0;
            }
            else
            {
                MessageBox.Show("Изменяйте ползунок над кнопкой, что бы выбрать, сколько нанять людей");
            }
            ArmyTrack();
            Update_Text();
        }

        private void HireSolder_Click(object sender, EventArgs e)
        {
            if (TrackBarSolder.Value > 0)
            {
                PeopleLive[City] -= TrackBarSolder.Value;
                GarSolder[City] += TrackBarSolder.Value;
                Money[lead] -= (PriceSolderGold[lead] * TrackBarSolder.Value);
                Weapon[lead] -= (PriceSolderWeapon[lead] * TrackBarSolder.Value);
                TrackBarSolder.Maximum -= TrackBarSolder.Value;
                TrackBarSolder.Value = 0;
            }
            else
            {
                MessageBox.Show("Изменяйте ползунок над кнопкой, что бы выбрать, сколько нанять людей");
            }
            ArmyTrack();
            Update_Text();
        }

        private void HireTank_Click(object sender, EventArgs e)
        {
            if (TrackBarTank.Value > 0)
            {
                PeopleLive[City] -= TrackBarTank.Value;
                GarTank[City] += TrackBarTank.Value;
                Money[lead] -= (PriceTankGold[lead] * TrackBarTank.Value);
                Tank[lead] -= (PriceTankTank[lead] * TrackBarTank.Value);
                TrackBarTank.Maximum -= TrackBarTank.Value;
                TrackBarTank.Value = 0;
            }
            else
            {
                MessageBox.Show("Изменяйте ползунок над кнопкой, что бы выбрать, сколько нанять людей");
            }
            ArmyTrack();
            Update_Text();
        }

        private void TrackBarSolder_Scroll(object sender, EventArgs e)
        {
            Update_Text();
        }

        private void TrackBarTank_Scroll(object sender, EventArgs e)
        {
            Update_Text();
        }

        private void CityPromManager_Click(object sender, EventArgs e)
        {
            Close_Menu();
            CityMenuBack.Visible = true;
            LiveShow.Visible = true;
            WhoControlTown.Visible = true;
            LaberCity.Visible = true;
            BuildFactory.Visible = true;
            BuildFactoryTank.Visible = true;
            BuildFactoryWeapon.Visible = true;
            HowFactory.Visible = true;
            HowFactoryTank.Visible = true;
            HowFactoryWeapon.Visible = true;
            MenuBack.Visible = true;
            if (CFactory[City] == 10)
            {
                HowFactory.Text = "Построено " + CFactory[City] + " из 10. Достигнут максимум";
            }
            else
            {
                HowFactory.Text = "Построено " + CFactory[City] + " из 10. Цена: " + ((CFactory[City] * 100) + 100);
            }
            if (CFactoryWeapon[City] == 10)
            {
                HowFactoryWeapon.Text = "Построено " + CFactoryWeapon[City] + " из 10. Достигнут максимум";
            }
            else
            {
                HowFactoryWeapon.Text = "Построено " + CFactoryWeapon[City] + " из 10. Цена: " + ((CFactoryWeapon[City] * 150) + 150);
            }
            if (CFactoryTank[City] == 10)
            {
                HowFactoryTank.Text = "Построено " + CFactoryTank[City] + " из 10. Достигнут максимум";
            }
            else
            {
                HowFactoryTank.Text = "Построено " + CFactoryTank[City] + " из 10. Цена: " + ((CFactoryTank[City] * 200) + 300);
            }
        }

        private void BuildFactory_Click(object sender, EventArgs e)
        {
            if (CFactory[City] <= 9)
            {
                if (Money[lead] >= ((CFactory[City] * 100) + 100))
                {
                    Money[lead] -= ((CFactory[City] * 100) + 100);
                    CFactory[City]++;
                    if (CFactory[City] == 10)
                    {
                        HowFactory.Text = "Построено " + CFactory[City] + " из 10. Достигнут максимум";
                    }
                    else
                    {
                        HowFactory.Text = "Построено " + CFactory[City] + " из 10. Цена: " + ((CFactory[City] * 100) + 100);
                    }
                    Update_Text();
                }
            }
            else
            {
                MessageBox.Show("Нельзя построить больше 10 заводов в одном поселении");
            }
        }

        private void BuildFactoryWeapon_Click(object sender, EventArgs e)
        {
            if (CFactoryWeapon[City] <= 9)
            {
                if (Money[lead] >= ((CFactoryWeapon[City] * 150) + 150))
                {
                    Money[lead] -= ((CFactoryWeapon[City] * 150) + 150);
                    CFactoryWeapon[City]++;
                    if (CFactoryWeapon[City] == 10)
                    {
                        HowFactoryWeapon.Text = "Построено " + CFactoryWeapon[City] + " из 10. Достигнут максимум";
                    }
                    else
                    {
                        HowFactoryWeapon.Text = "Построено " + CFactoryWeapon[City] + " из 10. Цена: " + ((CFactoryWeapon[City] * 150) + 150);
                    }
                    Update_Text();
                }
            }
            else
            {
                MessageBox.Show("Нельзя построить больше 10 заводов в одном поселении");
            }
        }

        private void BuildFactoryTank_Click(object sender, EventArgs e)
        {
            if (CFactoryTank[City] <= 9)
            {
                if (Money[lead] >= ((CFactoryTank[City] * 200) + 300))
                {
                    Money[lead] -= ((CFactoryTank[City] * 200) + 300);
                    CFactoryTank[City]++;
                    if (CFactoryTank[City] == 10)
                    {
                        HowFactoryTank.Text = "Построено " + CFactoryTank[City] + " из 10. Достигнут максимум";
                    }
                    else
                    {
                        HowFactoryTank.Text = "Построено " + CFactoryTank[City] + " из 10. Цена: " + ((CFactoryTank[City] * 200) + 300);
                    }
                    Update_Text();
                }
            }
            else
            {
                MessageBox.Show("Нельзя построить больше 10 заводов в одном поселении");
            }
        }

        private void CityCharge_Click(object sender, EventArgs e)
        {
            Close_Menu();
            CityMenuBack.Visible = true;
            LiveShow.Visible = true;
            WhoControlTown.Visible = true;
            LaberCity.Visible = true;
            HowChargePeasant.Visible = true;
            HowChargeSolder.Visible = true;
            HowChargeTank.Visible = true;
            TrackChargePeasant.Visible = true;
            TrackChargeSolder.Visible = true;
            TrackChargeTank.Visible = true;
            MenuBack.Visible = true;
            ChargeUpdate();
            if (CPath1[City] >= 0)
            {
                Charge1.Visible = true;
                Charge1.Text = "Напасть на " + CityName[CPath1[City]];
            }
            if (CPath2[City] >= 0)
            {
                Charge2.Visible = true;
                Charge2.Text = "Напасть на " + CityName[CPath2[City]];
            }
            if (CPath3[City] >= 0)
            {
                Charge3.Visible = true;
                Charge3.Text = "Напасть на " + CityName[CPath3[City]];
            }
            if (CPath4[City] >= 0)
            {
                Charge4.Visible = true;
                Charge4.Text = "Напасть на " + CityName[CPath4[City]];
            }
            if (CPath5[City] >= 0)
            {
                Charge5.Visible = true;
                Charge5.Text = "Напасть на " + CityName[CPath5[City]];
            }
        }

        private void Charge1_Click(object sender, EventArgs e)
        {
            if (TrackChargePeasant.Value != 0 || TrackChargeSolder.Value != 0 || TrackChargeTank.Value != 0)
            {
                CityTarget = CPath1[City];
                System.Console.WriteLine("Целью установлен " + CityName[CityTarget] + " с айди " + CPath1[City]);
                TryMoveToCity();
                ChargeUpdate();
                Update_Text();
            }
            else
            {
                MessageBox.Show("Выберите, сколько солдат пойдёт в атаку");
            }
        }

        public void BotTryMoveToCity()
        {
            leadAtk = CityOwner[CityBot];
            if (CityOwner[CityTarget] == leadAtk || (GarPeasant[CityTarget] == 0 && GarSolder[CityTarget] == 0 && GarTank[CityTarget] == 0))
            {
                if (CityOwner[CityTarget] != leadAtk)
                {
                    Text = "Город " + CityName[CityTarget] + " без боя захватила армия " + NameLead[leadAtk] + ".";
                    NewsNew();
                }
                if (CityOwner[CityTarget] == lead)
                {
                    // MessageBox.Show("В вашем городе " + CityName[CityTarget] + " не было гарнизона и войска " + NameLead[CityOwner[CityBot]] + " взяли его без боя");
                }
                CityOwner[CityTarget] = CityOwner[CityBot];
                GarPeasant[CityTarget] += Convert.ToInt32(GarPeasant[CityBot] * 0.5);
                GarPeasant[CityBot] = Convert.ToInt32(GarPeasant[CityBot] * 0.5);
                GarSolder[CityTarget] += Convert.ToInt32(GarSolder[CityBot] * 0.5);
                GarSolder[CityBot] = Convert.ToInt32(GarSolder[CityBot] * 0.5);
                GarTank[CityTarget] += Convert.ToInt32(GarTank[CityBot] * 0.5);
                GarTank[CityBot] = Convert.ToInt32(GarTank[CityBot] * 0.5);
                Victory();
                if (City == CityTarget)
                {
                    Close_Menu();
                }
            }
            else
            {
                AtkNum[1] = Convert.ToInt32(GarPeasant[CityBot] * 0.5);
                AtkNum[2] = Convert.ToInt32(GarSolder[CityBot] * 0.5);
                AtkNum[3] = Convert.ToInt32(GarTank[CityBot] * 0.5);
                System.Console.WriteLine("Бот готов драться, у него армия размерами " + AtkNum[1] + " " + AtkNum[2] + " " + AtkNum[3]);
                Battle();
                GarPeasant[CityBot] = Convert.ToInt32(GarPeasant[CityBot] * 0.5);
                GarSolder[CityBot] = Convert.ToInt32(GarSolder[CityBot] * 0.5);
                GarTank[CityBot] = Convert.ToInt32(GarTank[CityBot] * 0.5);
            }
            leadAtk = 0;
        }

        public void TryMoveToCity()
        {
            leadAtk = CityOwner[City];
            if (CityOwner[CityTarget] == lead || (GarPeasant[CityTarget] == 0 && GarSolder[CityTarget] == 0 && GarTank[CityTarget] == 0))
            {
                Text = "Город " + CityName[CityTarget] + " захватила армия " + NameLead[leadAtk] + ".";
                if (CityOwner[CityTarget] != lead)
                {
                    NewsNew();
                    CityOwner[CityTarget] = lead;
                    // MessageBox.Show("В городе " + CityName[CityTarget] + " не было гарнизона, вы взяли его без боя");
                }
                GarPeasant[CityTarget] += TrackChargePeasant.Value;
                GarPeasant[City] -= TrackChargePeasant.Value;
                GarSolder[CityTarget] += TrackChargeSolder.Value;
                GarSolder[City] -= TrackChargeSolder.Value;
                GarTank[CityTarget] += TrackChargeTank.Value;
                GarTank[City] -= TrackChargeTank.Value;
                Victory();
            }
            else
            {
                AtkNum[1] = TrackChargePeasant.Value;
                AtkNum[2] = TrackChargeSolder.Value;
                AtkNum[3] = TrackChargeTank.Value;
                System.Console.WriteLine("" + AtkNum[1] + " " + AtkNum[2] + " " + AtkNum[3]);
                Battle();
                GarTank[City] -= TrackChargeTank.Value;
                GarSolder[City] -= TrackChargeSolder.Value;
                GarPeasant[City] -= TrackChargePeasant.Value;
            }
        }

        public void Battle()
        {
            // Не забудь, что AtkNum отдельно записывается перед боем и для "ботов" нужно будет ещё прописать доп. строчки
            int[] DefNum = { 0, GarPeasant[CityTarget], GarSolder[CityTarget], GarTank[CityTarget] };
            int SumAtk = AtkNum[1] + AtkNum[2] + AtkNum[3];
            AtkNum[4] = AtkNum[1]; AtkNum[5] = AtkNum[2]; AtkNum[6] = AtkNum[3]; //Запоминает состав атакующих до битвы, что бы потом подсчитать потери
            int SumDef = DefNum[1] + DefNum[2] + DefNum[3];
            byte TypeA;
            byte TypeD; //У меня лапки, не бей за столь кривую боёвку
            int NumA;
            int NumD;
            bool Ph = false; //Фаза боя. true - атакующие атакуют, false - контратака
            bool Vic = false;
            System.Console.WriteLine("Предцикловые настройки");
            for (int i = 0; i < 99999; i++)
            {
                System.Console.WriteLine("Цикл сработал");
                if (Ph == true)
                {
                    Ph = false;
                    System.Console.WriteLine("Фаза защиты");
                }
                else
                {
                    Ph = true;
                    System.Console.WriteLine("Фаза атаки");
                }

                System.Console.WriteLine("Кол атакующих " + SumAtk);
                System.Console.WriteLine("Кол защищающихся " + SumDef);

                int A = rnd.Next(1, 4);
                int D = rnd.Next(1, 4);
                if (SumAtk > 0 && SumDef > 0) // В этом условии мы случайно выбираем того, какой тип войск атакует и записываем его айдишник с силой
                {
                    System.Console.WriteLine("Выбираем атакующий тип");
                Fucc:
                    if (AtkNum[A] > 0)
                    {
                        goto Gud;
                    }
                    else
                    {
                        A++;
                        if (A > 3)
                        {
                            A = 1;
                        }
                        goto Fucc;
                    }
                Gud:
                    System.Console.WriteLine("Выбрали атакующий тип");
                    TypeA = Convert.ToByte(A);
                    NumA = AtkNum[A];
                }
                else
                {
                    if (SumAtk < 0)
                    {
                        System.Console.WriteLine("Все атакующие мертвы, победа!");
                        Vic = false; //Срабатывает если армия мертва в нулину
                        goto TheEnd;
                    }
                    else
                    {
                        System.Console.WriteLine("Все защитники мертвы, победа!");
                        Vic = true;
                        goto TheEnd;
                    }
                }

                if (SumDef > 0 && SumAtk > 0) //Проворачиваем тоже самое и выбираем "обороняющийся отряд"
                {
                    System.Console.WriteLine("Выбираем защищающийся тип");
                Fucc:
                    if (DefNum[D] > 0)
                    {
                        goto Gud;
                    }
                    else
                    {
                        D++;
                        if (D > 3)
                        {
                            D = 1;
                        }
                        goto Fucc;
                    }
                Gud:
                    System.Console.WriteLine("Выбрали защищающийся тип");
                    TypeD = Convert.ToByte(D);
                    NumD = DefNum[D];
                }
                else
                {
                    if (SumAtk <= 0)
                    {
                        System.Console.WriteLine("Все атакующие мертвы, победа!");
                        Vic = false; //Срабатывает если армия мертва в нулину
                        goto TheEnd;
                    }
                    else
                    {
                        System.Console.WriteLine("Все защитники мертвы, победа!");
                        Vic = true;
                        goto TheEnd;
                    }
                }

                //Type = у нас записан айди армии, в Num = их численность
                // Теперь сам бой (зависит от фазы)

                double AtkBon = 1; //Бонус для атакующих в зависимости от типа. 
                double DefBon = 1; //Бонус для защищающихся в зависимости от типа
                if (Ph == true)
                {
                    System.Console.WriteLine("Ураа блять в атаку нахуй!");
                    if (A == 1) { AtkBon = 1; }    if (A == 2) { AtkBon = 0.8; }  if (A == 3) { AtkBon = 1.4; }
                    if (D == 1) { DefBon = 1; }    if (D == 2) { DefBon = 1.2; }  if (D == 3) { DefBon = 0.7; }
                    if (AtkNum[TypeA] > 0)
                    {
                        DefNum[TypeD] -= Convert.ToInt32(AtkNum[TypeA] * AtkBon * (2 - DefBon) * (1 - (Fort[CityTarget] * 0.1)) * 0.5);
                    }
                    if (DefNum[TypeD] > 0)
                    {
                        AtkNum[TypeA] -= Convert.ToInt32(DefNum[TypeD] * 0.25 * (2 - AtkBon));
                    }
                    System.Console.WriteLine("Потери атакующих на данном этапе боя: Ополчение: " + (TrackChargePeasant.Value - AtkNum[1]) + " чел. Солдат: " + (TrackChargeSolder.Value - AtkNum[2]) + " чел. Танки: " + (TrackChargeTank.Value - AtkNum[3]) + " чел.");
                }
                else
                {
                    System.Console.WriteLine("Ухх сука бля держим оборону");
                    if (D == 1) { DefBon = 1; }  if (D == 2) { DefBon = 0.8; }   if (D == 3) { DefBon = 1.4; }
                    if (A == 1) { AtkBon = 1; }  if (A == 2) { AtkBon = 1.2; }   if (A == 3) { AtkBon = 0.7; }
                    if (DefNum[TypeD] > 0)
                    {
                        AtkNum[TypeA] -= Convert.ToInt32(DefNum[TypeD] * (2 - AtkBon) * DefBon * 0.5);
                    }
                    if (AtkNum[TypeA] > 0)
                    {
                        DefNum[TypeD] -= Convert.ToInt32(AtkNum[TypeA] * 0.25 * (2 - DefBon) * (1 - (Fort[CityTarget] * 0.1)));
                    }
                    System.Console.WriteLine("Потери атакующих на данном этапе боя: Ополчение: " + (TrackChargePeasant.Value - AtkNum[1]) + " чел. Солдат: " + (TrackChargeSolder.Value - AtkNum[2]) + " чел. Танки: " + (TrackChargeTank.Value - AtkNum[3]) + " чел.");
                }
                for (int Y = 0; Y < 3; Y++)
                {
                    if (AtkNum[Y] < 1)
                    {
                        AtkNum[Y] = 0;
                    }
                    if (DefNum[Y] < 1)
                    {
                        DefNum[Y] = 0;
                    }
                }
                SumAtk = AtkNum[1] + AtkNum[2] + AtkNum[3];
                SumDef = DefNum[1] + DefNum[2] + DefNum[3];
            }
        TheEnd:
            System.Console.WriteLine("Конец боя, подсчитываем результат");
            //Vic = 2 если выиграли защитники, Vic = 1 если выиграли атакующие
            if (Vic == true)
            {
                if (leadAtk == lead)
                {
                    MessageBox.Show("Вы захватили " + CityName[CityTarget] + ", Потери:\nКрестьян: " + (AtkNum[4] - AtkNum[1]) + " чел.\nСолдат: " + (AtkNum[5] - AtkNum[2]) + " чел.\nТанки: " + (AtkNum[6] - AtkNum[3]));
                }
                for (int i = 0; i < 3; i++) { if (AtkNum[i] <= 0) { AtkNum[i] = 0; } }
                GarPeasant[CityTarget] = AtkNum[1];
                GarSolder[CityTarget] = AtkNum[2];
                GarTank[CityTarget] = AtkNum[3];
                Text = "Город " + CityName[CityTarget] + " захватила армия " + NameLead[leadAtk] + ".";
                NewsNew();
                CityOwner[CityTarget] = leadAtk;
                if (City == CityTarget)
                {
                    Close_Menu();
                }
            }
            else
            {
                if (leadAtk == lead)
                {
                    MessageBox.Show("Атака на " + CityName[CityTarget] + " не удалась, Потери врага:\nКрестьян: " + (GarPeasant[CityTarget] - DefNum[1]) + " чел.\nСолдат: " + (GarSolder[CityTarget] - DefNum[2]) + " чел.\nТанки: " + (GarTank[CityTarget] - DefNum[3]));
                }
                if (CityOwner[CityTarget] == lead)
                {
                    Text = "Ваш город " + CityName[CityTarget] + " попытался захватить " + NameLead[leadAtk] + ". Неудачно!";
                    NewsNew();
                }
                for (int i = 0; i < 3; i++) { if (DefNum[i] <= 0) { DefNum[i] = 0; } }
                GarPeasant[CityTarget] = DefNum[1];
                GarSolder[CityTarget] = DefNum[2];
                GarTank[CityTarget] = DefNum[3];
            }
            Victory();
            leadAtk = 0;
            CityTarget = 0;
        }

        private void Charge2_Click(object sender, EventArgs e)
        {
            if (TrackChargePeasant.Value != 0 || TrackChargeSolder.Value != 0 || TrackChargeTank.Value != 0)
            {
                CityTarget = CPath2[City];
                System.Console.WriteLine("Целью установлен " + CityName[CityTarget] + " с айди " + CPath2[City]);
                TryMoveToCity();
                ChargeUpdate();
                Update_Text();
            }
            else
            {
                MessageBox.Show("Выберите, сколько солдат пойдёт в атаку");
            }
        }

        private void Charge3_Click(object sender, EventArgs e)
        {
            if (TrackChargePeasant.Value != 0 || TrackChargeSolder.Value != 0 || TrackChargeTank.Value != 0)
            {
                CityTarget = CPath3[City];
                TryMoveToCity();
                ChargeUpdate();
                Update_Text();
            }
            else
            {
                MessageBox.Show("Выберите, сколько солдат пойдёт в атаку");
            }
        }

        private void Charge4_Click(object sender, EventArgs e)
        {
            if (TrackChargePeasant.Value != 0 || TrackChargeSolder.Value != 0 || TrackChargeTank.Value != 0)
            {
                CityTarget = CPath4[City];
                TryMoveToCity();
                ChargeUpdate();
                Update_Text();
            }
            else
            {
                MessageBox.Show("Выберите, сколько солдат пойдёт в атаку");
            }
        }

        private void Charge5_Click(object sender, EventArgs e)
        {
            if (TrackChargePeasant.Value != 0 || TrackChargeSolder.Value != 0 || TrackChargeTank.Value != 0)
            {
                CityTarget = CPath5[City];
                TryMoveToCity();
                ChargeUpdate();
                Update_Text();
            }
            else
            {
                MessageBox.Show("Выберите, сколько солдат пойдёт в атаку");
            }
        }

        public void ChargeUpdate()
        {
            TrackChargePeasant.Maximum = GarPeasant[City];
            TrackChargeSolder.Maximum = GarSolder[City];
            TrackChargeTank.Maximum = GarTank[City];
            TrackChargeTank.Value = TrackChargeTank.Minimum;
            TrackChargeSolder.Value = TrackChargeSolder.Minimum;
            TrackChargePeasant.Value = TrackChargePeasant.Minimum;
        }

        private void TrackChargePeasant_Scroll(object sender, EventArgs e)
        {
            Update_Text();
        }

        private void TrackChargeSolder_Scroll(object sender, EventArgs e)
        {
            Update_Text();
        }

        private void TrackChargeTank_Scroll(object sender, EventArgs e)
        {
            Update_Text();
        }

        public void Update_Text()
        {
            MoneyShow.Text = "" + Money[lead];
            WeaponShow.Text = "" + Weapon[lead];
            TankShow.Text = "" + Tank[lead];
            HowChargeTank.Text = "Отправить " + TrackChargeTank.Value + " танков из " + TrackChargeTank.Maximum;
            HowChargeSolder.Text = "Отправить " + TrackChargeSolder.Value + " солдат из " + TrackChargeSolder.Maximum;
            HowChargePeasant.Text = "Отправить " + TrackChargePeasant.Value + " ополченцев из " + TrackChargePeasant.Maximum;
            HowManyTank.Text = "Призвать " + TrackBarTank.Value + " танков из " + TrackBarTank.Maximum;
            HowManySolder.Text = "Призвать " + TrackBarSolder.Value + " солдат из " + TrackBarSolder.Maximum;
            HowManyPeasant.Text = "Призвать " + TrackBarPeasant.Value + " ополченцев из " + TrackBarPeasant.Maximum;
            if (City >= 0)
            {
                LiveShow.Text = "Население: " + PeopleLive[City] + " чел.";
                ArmyStatusCity.Text = "Статус армии:\nОполченцев: " + GarPeasant[City] + "\nСолдат: " + GarSolder[City] + "\nТанков: " + GarTank[City];
                LaberCity.Text = "Город " + CityName[City];
                DestroyFactoryText.Text = "В городе есть " + CFactory[City] + " заводов";
                DestroyWeaponText.Text = "В городе есть " + CFactoryWeapon[City] + " оруж. заводов";
                DestroyTankText.Text = "В городе есть " + CFactoryTank[City] + " танк. заводов";
                DestroyFortText.Text = "В городе есть " + Fort[City] + " фортов";
                DestroyPeopleText.Text = "Вы можете ликвидировать " + (int)(PeopleLive[City] * 0.01) + " людей";
            }
            for (int i = 0; i < Stab.Length; i++)
            {
                if (Stab[i] < 0)
                {
                    Stab[i] = 0;
                }
            }
        }

        private void TimerPicture_Click(object sender, EventArgs e)
        {
            if (RealTimeOn == true)
            {
                Sec10.Stop();
                RealTimeOn = false;
                TimerPicture.BackgroundImage = Properties.Resources.TimeStop;
            }
            else
            {
                Sec10.Start();
                RealTimeOn = true;
                TimerPicture.BackgroundImage = Properties.Resources.TimePlay;
            }
        }

        private void NewsOpen_Click(object sender, EventArgs e)
        {
            NewsPanel.Visible = true;
            NewsOpen.Visible = false;
        }

        public void NewsNew()
        {
            News16.Text = News15.Text;
            News16.ForeColor = News15.ForeColor;
            News15.Text = News14.Text;
            News15.ForeColor = News14.ForeColor;
            News14.Text = News13.Text;
            News14.ForeColor = News13.ForeColor;
            News13.Text = News12.Text;
            News13.ForeColor = News12.ForeColor;
            News12.Text = News11.Text;
            News12.ForeColor = News11.ForeColor;
            News11.Text = News10.Text;
            News11.ForeColor = News10.ForeColor;
            News10.Text = News9.Text;
            News10.ForeColor = News9.ForeColor;
            News9.Text = News8.Text;
            News9.ForeColor = News8.ForeColor;
            News8.Text = News7.Text;
            News8.ForeColor = News7.ForeColor;
            News7.Text = News6.Text;
            News7.ForeColor = News6.ForeColor;
            News6.Text = News5.Text;
            News6.ForeColor = News5.ForeColor;
            News5.Text = News4.Text;
            News5.ForeColor = News4.ForeColor;
            News4.Text = News3.Text;
            News4.ForeColor = News3.ForeColor;
            News3.Text = News2.Text;
            News3.ForeColor = News2.ForeColor;
            News2.Text = News1.Text;
            News2.ForeColor = News1.ForeColor;
            News1.Text = Text;
            News1.ForeColor = System.Drawing.Color.Black;
            if (CityOwner[CityTarget] == lead)
            {
                News1.ForeColor = System.Drawing.Color.Red;
            }
            if (leadAtk == lead)
            {
                News1.ForeColor = System.Drawing.Color.Green;
            }
        }

        private void NewsClose_Click(object sender, EventArgs e)
        {
            NewsPanel.Visible = false;
            NewsOpen.Visible = true;
            NewsOpen.BringToFront();
        }

        public void Victory()
        {
            System.Console.WriteLine("Проверка победы");
            bool N = true;
            N = true;
            for (int i = 0; i < CityOwner.Length; i++)
            {
                if (CityOwner[i] != CityOwner[0])
                {
                    System.Console.WriteLine("Нет, у тебя нету всех городов, ты не достоин победы!");
                    N = false;
                }
            }
            if (N == true)
            {
                TimerPicture.Visible = false;
                System.Console.WriteLine("ПОЛНАЯ ПОБЕДА ПОЛНАЯ ПОБЕДА ПОЛНАЯ ПОБЕДА");
                RealTimeOn = false;
                Sec10.Stop();
                VictoryScreen.Location = new Point(100, 100);
                VictoryScreen.Visible = true;
                VictoryLeader.BackgroundImage = LeadPortret[CityOwner[0]];
                if (CityOwner[0] == 0)
                {
                    VictoryLeader.BackgroundImage = Properties.Resources.Lyka;
                    VictoryIdeo.BackgroundImage = Properties.Resources.Ideo1;
                    VictoryText.Text = "Лукашенко чудесным образом смог вернутся на \nместо диктатора Беларуси.Хоть он и был причиной \nвсего этого конфликта, но его армия оказалась \nсамой сильной и организованной в этом конфликте. \nБеларусь ждала судьба Северной Кореи на несколько \nлет, пока после его смерти сына-Колю не свергло \nобъединение из военных, но это уже другая и \nдалёкая история... Вечна жыве и квiтней Беларусь!";
                }
                if (CityOwner[0] == 1)
                {
                    VictoryIdeo.BackgroundImage = Properties.Resources.Ideo1;
                    VictoryText.Text = "СНО смогло захватить власть в стране. Народная\nармия смогла отбросить из страны всю военную хунту\n и установить народовластие. Разрушенная страна\n какое то время ещё жила на ''идее единства'', но со\n временем различная политическая грязь, опираясь на\n бедственное экономическое положение и реваншизм\n некоторых социальных групп ввергнула \nстрану в новый политический кризис, но это уже\nдругая история...";
                }
                if (CityOwner[0] == 5)
                {
                    VictoryIdeo.BackgroundImage = Properties.Resources.Ideo1;
                    VictoryText.Text = "Равков смог перехватить власть в республике...\nОчередной военный диктатор не сулит ничего \nхорошего стране. Наступило время деспотизма и \nмилитаризма, однако сил Равкова не хватило, что \nбы удержать страну в стабильном состоянии. \nБанд-формирования и контрабанда стали \nвскакивать, как прыщи у подростка. Со временем \nБеларусь скатилась в Failed-state с кучей оружия.";
                }
            }
            Update_Point();
            System.Console.WriteLine("Проверка победы завершилась");
        }

        private void NewsSmol_Click(object sender, EventArgs e)
        {
            if (NewsPanel.Size.Height == 310)
            {
                NewsPanel.Size = new Size(470, 160);
                NewsPanel.Location = new Point(535, 560);
            }
            else
            {
                NewsPanel.Size = new Size(470, 310);
                NewsPanel.Location = new Point(535, 410);
            }
        }

        public void EventShutdown()
        {
            EventPanel.Visible = false;
            EventPic.Visible = false;
            EventButt1.Visible = false;
            EventButt2.Visible = false;
            EventButt3.Visible = false;
            EventButt4.Visible = false;
            EventButt5.Visible = false;
            EventButt6.Visible = false;
            EventText.Visible = false;
            TimerPicture.Visible = true;
            TimerPicture.BackgroundImage = Properties.Resources.TimeStop;
        }

        public void EventLoad()
        {
            EventPanel.Location = new Point(150, 50);
            if (Eid == 1)
            {
                EventButt1.Visible = true;
                EventButt2.Visible = true;
                EventButt1.Text = "Собери оставшихся бойцов";
                EventButt2.Text = "Дайте мне листок и ручку";
                EventText.Text = "Михаил Гриб спустился по ступенькам в штаб обороны Минска, где уже которую неделю сидел Лукашенко и рассуждал, что ему делать. \nОн не церемонился. Без приветствий и какого то уважения просто поставил перед фактом: Полноценная атака города начнётся \nв течении дня. Лукашенко, и без того мрачный, потерял в красках ещё сильнее - как себя защитить? Речь уже даже идёт не об \nудержании города, а о хотя бы возможности его покинуть живым и где то залечь на дно, желательно с деньгами. Но куда?\nОдно Лукашенко знал точно - город он не покинет, даже если хочет...";
                CanFocus[1] = true;
                FocusInfo[1] = "Сделать зелёненьким\nНам надоело, что фон этой кнопки - красный. Мы хотим нажать на неё и поменять цвет, типо \nфокус выполнен...";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 2)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "Анархия мама сынов своих любит!";
                EventText.Text = "Мэр Пинска наконец то сбежал! Лишь пять полицейских забарикадировались в мэрии и на что то надеялись против наших ребят. \nРеволюция анархистов в отдельно взятом городе обернулась успехом - Пинск наш, но топтаться на месте мы не собираемся и у\nнас есть оружие. Хоть Че Гевара был коммунистом - его теория революции была права: достаточно лишь группы правильно\nнастроенных радикальнх революционеров, что бы устроить революцию, не дожидаясь ''созревания'' общества.";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 3)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "Веди нас, министр обороны!";
                EventText.Text = "Телефон в Витебской военной части не звонит уже сутки. До этого он разрывался от звонков с надеждой, что кто то поднимет\nтрубку. Но Равков уже выбрал сторону - военная хунта, которую он возглавит. К слову, не сильно отличается от 'официальной'\nвласти, в Беларуси сейчас мало кого можно назвать легитимным. В Минске, кажется, теперь осознали бесмыссленость сие\nдействия.";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 4)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "Нас ждут тяжёлые времена...";
                EventText.Text = "Над Гродно развивался бело-красно-белый флаг с чёрным сжатым кулаком посредине. За 2 дня СНО (силы народной обороны) \nи подумать не могли, что восстание, подобное Гродненскому, вспыхнет за пару недель по всей стране, но с абсолютно разными\nидеологиями, целями и последствиями (по 'остаткам' связи дошла информация, что Минск остался за Лукашенко). Мы пока не\nпринимали попыток связаться с соседними поселениями, где так же смогли вытеснить официальные власти, но судя по общему \nхаосу - на успех лучше не расчитывать. Такое чувство, что никто даже не был готов к тому, что бы победить. Многие 'мужики'\nбуквально штурмовали квартал за кварталом при помощи арматуры, ножей и бит. Это была самоубийственная и отчаянная битва\nкоторую мы выиграли. Но что теперь?";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 5)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "Ну, подорвали пацаны!";
                EventText.Text = "Орша ровном месяц находится в состоянии оккупации местным ОПГ. Этот город в восточной Беларуси и до этого имел славу одного\nиз самых криминальных городов в Беларуси, а теперь окончательно закрепил этот титул за собой. Роман (бандитская кличка -  \nШишак) сидел в местом доме культуры и развлекался с Оршанскими проститутками, но радость от уединения прервало уведомление.\nСообщение от подчинённого было со скриншотом новостного заголовка об начале битвы за Минск. Кажется, Беларусь окончательно\nвпала в состояние гражданской войны и плацдарм Орши между Могилёвом и Витебском явно не оставят нетронутым. Как известно, \nв Витебске сейчас сидит часть бывшей военной верхушки, и они, обладая танками и оружием попытаются вернуть контроль над \nОршей, от которой помимо пути к Могилёву ведёт железная дорога прямиком в Минск. Шишак попросил девочек удалится и начал\nнабирать номер главы местного ополчения. Надо поднимать людей и быть готовыми к хлопотному дельцу...";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 6)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "Помолимся за Беларусь!";
                EventText.Text = "Это было быстро. Васькович вместе с сотней верных штурмовиков ''Молодого фронта'' ворвался в Полоцкую исправительную\nколонию и, завязав недлительный бой, захватил её. Заключённые были шокированы и даже не все вышли из камер, думая, что это\nчто это какая-то провокация и постановка, но Васьковича они всё равно не интересовали. Наконец то были освобождены его\nлучшие люди, включая Павла, Ярослава, Витаутаса и десяток других бойцов. Через пол часа, по приходу в Полоцк, он узнал,\nсамопровозглашённый президент независимого Полоцка сбежал! В целом, сейчас чуть ли не каждая деревня нашла себе своего \nварлорда и 'объявила независимость'. Глупости. Беларусь не должна страдать от раздробленности, наша родина не заслужила\nэтого. Мы обладаем моноэтническим населением, у нас хороший фундамент что бы построить националистический 'Белый рай',\nЕдиную Белую Русь! И кто это сделает, кроме нас?";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 7)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "Мы не допустим захвата Бреста!";
                EventButt2.Visible = true;
                EventButt2.Text = "Деспотизм Минска должен уступить место новой коммунистической революции";
                EventButt3.Visible = true;
                EventButt3.Text = "Мы объявим себя наследниками БССР как истинной Беларуси";
                EventButt4.Visible = true;
                EventButt4.Text = "Главное - это стабильность и жизнь народа";
                EventText.Text = "Брест пережил первые дни Беларуского хаоса спокойно относительно остальных областных городов. В городе на момент начала\nконфликта сильные и, что важно, легитимные позиции имела местная марионеточная коммунистическая партия во главе с Алексеем.\nОн проявил сообразительность и в течении месяца аккуратно балансировал между указами из Минска и 'волей народа'. Задавив \nлишь самые радикальные и опасные элементы он обеспечил город продовольствием, связью и прочими благами цивилизации. Но \nвремя спокойствия и стабильности уходит. Люди всё больше напряжены тем, что творится в стране, а приказы из Минска просто\nабсурдные! Мы не собираемся стягивать все городские силы в столицу, формировать добровольцев, отправлять туда же еду и\nоружие, оно необходимо и нам тут. До Алексея дошли новости о начале по всей стране вооружённых стычек, нам необходимо\nвыходить из зоны комфорта. Для начала, Брест и наша партия в частности должны будут объявить о своей 'независимости'. Но\nо какой именно? Автономный Брест? 'Настоящая' Беларусь? БССР? Более того, означает ли это, что мы тоже затянемся в эту\nгражданскую войну? Как все эти новости примут люди? Это тяжёлый выбор, но мы должны это сделать. Сегодня мы устроим \nсобрание на городской площади и произнесём важную для всех речь. Как мы объявим вступление в этот хаос?";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 8)
            {
                if (Money[lead] >= 50)
                {
                    EventButt2.Visible = true;
                    EventButt2.Text = "Проведём пышный банкет";
                    EventButt4.Visible = true;
                    EventButt4.Text = "Объявим выходной народу в честь этого";
                }
                EventButt1.Visible = true;
                EventButt1.Text = "У нас не так много средств, может, в этот раз будем скромнее?";
                if (Weapon[lead] >= 50 && Tank[lead] >= 5)
                {
                    EventButt3.Visible = true;
                    EventButt3.Text = "Военный парад в честь Лукашенко, УРА!";
                }
                EventText.Text = "30 Апреля Лукашенко отмечает свой очередной день рождения. Наш народный вождь не должен лишать себя радостей праздинка, \nкак мы будем отмечать это в условиях военного времени?";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 9)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "Наука спасёт Беларусь!";
                EventText.Text = "Гомель был охвачен хаосом, и с этим хаосом необходимо разбираться Чижикову и его совету. Группа технократов, ''победившая''\nна местных выборах должна была решать, как успокоить город. Различные группировки в городе хотят перехватить власть у \nгруппы технократов. Самые сильные среди них - местная коммунистическая ячейка. Чижиков уже успел несколько раз продумать \nвариант, что бы добровольно сложить полномочия и сдать город хоть кому нибудь, а самому сбежать через Украину на запад. Но\nсовет, принципы (и возможно личная жажда власти) пока заставила его держаться в руководящем кресле. Решение было принято\nи группы верных Чижикову солдат начали патрулировать улицы. В Гомеле было введено военное положение, фитиль пороховой бочки\nбыл на время потушен, но эта бочка стоит в центре горящего склада, и лишь вопрос времени - когда она рванёт. Чижиков понимал,\nчто гражданской войны областному городу не избежать, и придётся воевать как минимум в пределах Гомельской области. Все \nумы Гомеля будут пущены на военные нужды, технократия будет создавать оружие и разрабатывать тактику, иначе нас задавят \nи силой, и числом...";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 10)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "К оружию!";
                EventText.Text = "Лида была одним из первых городов, где горожане сбросили оккупацию из Минска, причём сбросили самым суровым методом. \nЛиду избрали своей целью отколовшиеся от ''ОГСБ'' радикалы под лидерством Сацукевича. После захвата административных \nзданий, что далось очень нелегко, а в бою использовались мортиры и БТР, администрация была убита. Целиком. Все городские\nчиновники и выжившие солдаты-срочники с полицейскими были преданы ''народному суду'', всем до единого был вынесен смертный\nприговор. Надо осознавать ситуацию - даже горожане Лиды были в экстазе от этого спонтанного и нечеловеческого самосуда. \nОбстановка явно благоприятствовала, что бы даже такие отморозки нашли поддержку в народе. Чувство мести к властям превысило\nвсе какие либо здравые мысли, и Сацукевич использовал это. Организованное им временное правительство, которое по факту было\n''хунтой с поддержкой народа'' поставило чёткую цель - предать народному суду всех пособников режима, что довели страну до\nтакого состояния. Этот режим вынужден подпитывать сам себя чувством реваншизма, ему нужна война - и они её получат. Но как\nдолго их Лидское государство протянет лишь на идее реваншизма? Об этом никто не думает сейчас, Лидчанин должен убить\nкарателя режима! Смерть фашистам!";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 11)
            {
                EventButt1.Visible = true;
                EventButt1.Text = "Работаем братья";
                EventText.Text = "Когда революция всполыхнула с новой силой и Беларусь распалась на варлордию силовики поняли - дальше их безопасность лишь\nв их руках. Лукашенко в своём положении не защитит их, а даже если ''победит'', то большинство всё равно скорее всего сольют.\nКараев, направляющийся на юг беларуси для ''умировторения'' местных сил революции собрал кучку офицеров младшего ранга и \nпообещал им защиту от народного гнева, если они помогут ЛИЧНО ЕМУ установить свою власть над каким нибудь из Беларуских \nрегионов. Так и образовалась ещё одна воооружённая хунта в лице Осетина, которая держалась исключительно на страхе военных\nВ планах у Караева было максимально долго удерживать хоть какое то поселение, однако кроме как грабежом и без того бедных\nБеларуских городов он протянуть не сможет. Ирония ли, ведь самый безопасный вариант для Караева и его хунты был бы в том,\nчто бы сидеть тихо в городе и не зарабатывать себе больше военных преступлений. Это безумная авантюра погубит много людей...";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            if (Eid == 12)
            {
                if (TestFocus[1].BackColor == Color.Red)
                {
                    TestFocus[1].BackColor = Color.Green;
                }
                goto noshow;
            }
            if (Eid == 13)
            { 
                EventButt1.Visible = true;
                EventButt1.Text = "Пробудитесь, братья, пробудитесь, братья, пробудитесь, братья, с радостным сердцем!";
                EventText.Text = "Эта ночь в Сморгони была особенно необычной - над мэрией вместо старого красно-зелёного флага внезапно появилась иудейская\nзвезда, что стала сигналом для захвата местного участка и разоружении правоохранителей и силовиков. Почти вся небольшая\nЕврейская община собралась в одном Беларуском городе, в том числе их лидеры - Еврейский совет, собранный из лидеров общин и\nрелигиозных деятелей. Евреи помнили свой старый опыт. Практический все вооружённые конфликты, куда они были так или иначе\nвовлечены сопровождался Еврейскими погромами. Необходимо было организовать защиту всего Еврейского населения страны, дабы\nне повторить этот печальный опыт, но практический весь совет не собирался ограничиваться лишь защитой. Раз уж им предстояло\nвлезть в этот конфликт - можно попытаться построит тут второй Израиль и вернуть под контроль свои старые брошенные общины.\nВосточная Европа раньше славилась большим числом евреев до прихода одного ''усатого'' диктатора, так что амбиции общины \nбыли не безосновательные, но хватит ли им на это сил?";
                EventPic.BackgroundImage = Properties.Resources._21644;
            }
            Sec10.Stop();
            RealTimeOn = false;
            TimerPicture.Visible = false;
            EventPanel.Location = new Point(150, 50);
            EventPanel.Visible = true;
            EventPic.Visible = true;
            EventText.Visible = true;
        noshow:
            System.Console.WriteLine("пук");
        }

        private void EventButt1_Click(object sender, EventArgs e)
        {
            if (Eid == 1)
            {
                Stab[lead] += 3;
            }
            if (Eid == 4)
            {
                GarPeasant[2] += 60;
            }
            if (Eid == 7)
            {
                Fort[4] += 1;
            }
            if (Eid == 8)
            {
                Stab[lead] -= 2;
            }
            EventShutdown();
        }

        private void EventButt2_Click(object sender, EventArgs e)
        {
            if (Eid == 1)
            {
                GarSolder[1] += 50;
            }
            if (Eid == 7)
            {
                GarSolder[4] += 25;
            }
            if (Eid == 8)
            {
                Money[lead] -= 50;
            }
            EventShutdown();
        }

        private void EventButt3_Click(object sender, EventArgs e)
        {
            if (Eid == 8)
            {
                Weapon[lead] -= 50;
                Tank[lead] -= 5;
                Stab[lead] += 5;
            }
            EventShutdown();
        }

        private void EventButt4_Click(object sender, EventArgs e)
        {
            if (Eid == 7)
            {
                Stab[lead] += 4;
            }
            if (Eid == 8)
            {
                Money[lead] -= 50;
                Stab[lead] += 3;
            }
            EventShutdown();
        }

        private void EventButt5_Click(object sender, EventArgs e)
        {
            EventShutdown();
        }

        private void EventButt6_Click(object sender, EventArgs e)
        {
            EventShutdown();
        }

        private void MenuClose_Click(object sender, EventArgs e)
        {
            Close_Menu();
        }

        private void pictureBox2_Click(object sender, EventArgs e) // Это если что непереименнованная кнопочка "Назад" в меню города. Я боюсь её переименовывать, опять прога лагать начнёт и орать, что не может найти строчку
        {
            Close_Menu();
            CityMenuBack.Visible = true;
            LaberCity.Visible = true;
            if (CityOwner[City] == lead)
            {
                WhoControlTown.Text = "Это ваше поселение";
                CityArmyManager.Visible = true;
                CityPromManager.Visible = true;
                CityCharge.Visible = true;
                CityDestroy.Visible = true;
            }
            else
            {
                WhoControlTown.Text = "Этот город контролирует " + NameLead[CityOwner[City]];
            }
            WhoControlTown.Visible = true;
            MenuClose.BringToFront();
            Update_Text();
        }

        private void MenuButt_Click(object sender, EventArgs e)
        {
            MenuButt.Visible = false;
            FocusButt.Visible = false;
            MenuPanel.Visible = true;
            MenuStab.Text = "Стабильность: " + Stab[lead];
            MenuRecrut.Text = "Процент призывников: " + CanRecrut[lead];
            MenuLeaderName.Text = "" + NameLead[lead];
            LeaderPortret.BackgroundImage = LeadPortret[lead];
            MenuPanel.Location = new Point(500, 40);
        }

        private void CloseMenu_Click(object sender, EventArgs e)
        {
            MenuButt.Visible = true;
            FocusButt.Visible = true;
            MenuPanel.Visible = false;
        }

        private void FocusButt_Click(object sender, EventArgs e)
        {
            TestFocus[1] = Focus1; TestFocus[2] = Focus2; TestFocus[3] = Focus3; TestFocus[4] = Focus4; TestFocus[5] = Focus5; TestFocus[6] = Focus6; TestFocus[7] = Focus7; TestFocus[8] = Focus8; TestFocus[9] = Focus9; TestFocus[10] = Focus10; TestFocus[11] = Focus11; TestFocus[12] = Focus12; TestFocus[13] = Focus13; TestFocus[14] = Focus14; TestFocus[15] = Focus15; TestFocus[16] = Focus16;
            //Я правда очень пытался автоматизировать запись этого говна через цикл, но не понял как конвертировать string в picturebox
            FocusText.Visible = false;
            MenuButt.Visible = false;
            FocusButt.Visible = false;
            FocusPanel.Visible = true;
            FocusPanel.Location = new Point(500, 40);
            FocusMainName.Text = "Фокусы " + NameLead[lead];
            for (int i = 1; i < 17; i++)
            {
                if (CanFocus[i] == false)
                {
                    System.Console.WriteLine(i);
                    TestFocus[i].Visible = false;
                    System.Console.WriteLine("нет фокуса");
                }
                else
                {
                    TestFocus[i].Visible = true;
                    System.Console.WriteLine("есть фокус");
                }
            }    
        }

        private void Focus1_Click(object sender, EventArgs e)
        {
            if (TestFocus[1].BackColor == Color.Red)
            {
                switch (Convert.ToString(lead))
                {
                    case "0":
                        Eid = 12;
                        break;
                }
                EventLoad();
            }
        }
        private void Focus2_Click(object sender, EventArgs e)
        {

        }
        private void Focus3_Click(object sender, EventArgs e)
        {

        }
        private void Focus4_Click(object sender, EventArgs e)
        {

        }
        private void Focus5_Click(object sender, EventArgs e)
        {

        }
        private void Focus6_Click(object sender, EventArgs e)
        {

        }
        private void Focus7_Click(object sender, EventArgs e)
        {

        }
        private void Focus8_Click(object sender, EventArgs e)
        {

        }
        private void Focus9_Click(object sender, EventArgs e)
        {

        }
        private void Focus10_Click(object sender, EventArgs e)
        {

        }
        private void Focus11_Click(object sender, EventArgs e)
        {

        }
        private void Focus12_Click(object sender, EventArgs e)
        {

        }
        private void Focus13_Click(object sender, EventArgs e)
        {

        }
        private void Focus14_Click(object sender, EventArgs e)
        {

        }
        private void Focus15_Click(object sender, EventArgs e)
        {

        }
        private void Focus16_Click(object sender, EventArgs e)
        {

        }

        private void CloseFocus_Click(object sender, EventArgs e)
        {
            MenuButt.Visible = true;
            FocusButt.Visible = true;
            FocusPanel.Visible = false;
        }

        private void Focus1_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[1];
        }
        private void Focus2_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[2];
        }
        private void Focus3_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[3];
        }
        private void Focus4_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[4];
        }
        private void Focus5_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[5];
        }
        private void Focus6_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[6];
        }
        private void Focus7_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[7];
        }
        private void Focus8_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[8];
        }
        private void Focus9_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[9];
        }
        private void Focus10_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[10];
        }
        private void Focus11_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[11];
        }
        private void Focus12_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[12];
        }
        private void Focus13_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[13];
        }
        private void Focus14_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[14];
        }
        private void Focus15_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[15];
        }
        private void Focus16_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = true;
            FocusText.Text = FocusInfo[16];
        }

        private void FocusPanel_MouseEnter(object sender, EventArgs e)
        {
            FocusText.Visible = false;
        }

        private void CityDestroy_Click(object sender, EventArgs e)
        {
            Close_Menu();
            CityMenuBack.Visible = true;
            LiveShow.Visible = true;
            WhoControlTown.Visible = true;
            LaberCity.Visible = true;
            MenuBack.Visible = true;
            DestroyYesUCan.Visible = true;
            if (DestroyLimit[City] == false)
            {
                DestroyYesUCan.ForeColor = Color.Green;
                DestroyYesUCan.Text = "Вы можете сегодня что то уничтожить.";
                DestroyWeaponButt.Visible = true;
                DestroyTankButt.Visible = true;
                DestroyFactoryButt.Visible = true;
                DestroyPeopleButt.Visible = true;
                DestroyFortButt.Visible = true;
                DestroyFactoryText.Visible = true;
                DestroyFortText.Visible = true;
                DestroyWeaponText.Visible = true;
                DestroyTankText.Visible = true;
                DestroyPeopleText.Visible = true;
            }
            else
            {
                DestroyYesUCan.ForeColor = Color.Red;
                DestroyYesUCan.Text = "Вы сегодня исчерпали лимит разрушения\nв этом городе.";
            }
            Update_Text();
        }

        public void GetDestroy()
        {
            DestroyWeaponButt.Visible = false;
            DestroyTankButt.Visible = false;
            DestroyFactoryButt.Visible = false;
            DestroyFactoryText.Visible = false;
            DestroyFortText.Visible = false;
            DestroyWeaponText.Visible = false;
            DestroyTankText.Visible = false;
            DestroyPeopleText.Visible = false;
            DestroyFortButt.Visible = false;
            DestroyPeopleButt.Visible = false;
            DestroyYesUCan.ForeColor = Color.Red;
            DestroyYesUCan.Text = "Вы сегодня исчерпали лимит разрушения\nв этом городе.";
            DestroyLimit[City] = true;
        }

        private void DestroyFactoryButt_Click(object sender, EventArgs e)
        {
            if (CFactory[City] >= 1 && CityOwner[City] == lead)
            {
                Money[lead] += ((CFactory[City] * 50)+50);
                CFactory[City]--;
                GetDestroy();
            }
            Update_Text();
        }

        private void DestroyWeaponButt_Click(object sender, EventArgs e)
        {
            if (CFactoryWeapon[City] >= 1 && CityOwner[City] == lead)
            {
                Money[lead] += ((CFactoryWeapon[City] * 75)+50);
                CFactoryWeapon[City]--;
                GetDestroy();
            }
            Update_Text();
        }

        private void DestroyTankButt_Click(object sender, EventArgs e)
        {
            if (CFactoryTank[City] >= 1 && CityOwner[City] == lead)
            {
                Money[lead] += ((CFactoryTank[City] * 100)+50);
                CFactoryTank[City]--;
                GetDestroy();
            }
            Update_Text();
        }

        private void DestroyFortButt_Click(object sender, EventArgs e)
        {
            if (Fort[City] >= 1 && CityOwner[City] == lead)
            {
                Fort[City]--;
                GetDestroy();
            }
            Update_Text();
        }

        private void DestroyPeopleButt_Click(object sender, EventArgs e)
        {
            if (PeopleLive[City] >= 1 && CityOwner[City] == lead)
            {
                Stab[CityOwner[City]]--;
                PeopleLive[City] -= (int)(PeopleLive[City] * 0.01);
                GetDestroy();
            }
            Update_Text();
        }
    }
}
