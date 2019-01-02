# 中山大学数据科学与计算机学院本科生实验报告
## （2018年秋季学期）
| 课程名称 | 编译原理 | 任课老师 | 娄定俊 |
| :------------: | :-------------: | :------------: | :-------------: |
| 年级 | 16级 | 专业（方向） | 软件工程计应 |
| 学号 | 16340179 | 姓名 | 彭流生 |
| 电话 | 15989061655 | Email | 1728343423@qq.com |

### 一.	实验内容
1.	在计算机上实现PL0语言的编译程序;
2.	扩展PL0语言的功能,并在计算机上实现.

### 二.	什么是PL0语言?
1. PL0语言是PASCAL语言的子集, 是一个用于教学的模型语言;
2. PL0语言的语法图我们将发给大家;
3. PL0语言的编译程序是用PASCAL语言写的(我们将复印给大家);
4. PASCAL语言:
(1)	是瑞士计算机科学家N. Wirth为教学目的设计的;
(2)	是一个比较早期的程序设计语言(大约二十多年前), 与C语言历史差不多长;
(3)	它的特点是接近于自然语言(英语), 直观﹑易于理解.

### 三.	实验要做的工作
1. 找到PASCAL编译系统( Delphi系统也可以);
2. 在PASCAL系统上运行PL0编译程序,需要对PL0编译程序作一些修改﹑调试;
3. 在PASCAL系统中,为PL0的编译程序建立输入文件和输出文件;
 	在输入文件中存放PL0源程序(我们也复印给大家);
 	在输出文件中存放PL0源程序被编译后产生的中间代码和运行数据;
4. PL0的编译程序运行时, 通过输入文件输入PL0源程序, 在输出文件中产生源程序的中间代码, 然后运行该中间代码, 在输出文件中产生运行数据;
5. 如果上述工作成功, 则第一项实习任务完成.再做以下工作:
6. 在PL0语言中增加Read和Write语句;
7. 修改PL0编译程序, 使得PL0源程序可以使用Read和Write语句, 从文件(或键盘)输入数据,并可以向文件(或屏幕)写数据.
8. 用我们复印给大家的PL0源程序作为调试数据;
9. 若以上工作完成, 则第2项实验任务完成.

### 实验步骤
1. 安装PASCAL编译系统：`安装free pascal IDE`
2. 在命令行下编译修改后的pl0编译程序（假设文件名为compile.pas），主要命令为`fpc compile.pas`，效果图如下：  
   
   ![preview](https://github.com/penglsh/pl0/raw/master/images/compile.PNG)  
3. 在命令行下运行上述步骤生成的可执行文件compile.exe，进行程序的调试，效果图如下：  
   
   ![preview](https://github.com/penglsh/pl0/raw/master/images/run.PNG)  
4. 读入给定的pl0源程序，编译其后把产生的代码写入到输出文件中

### 第一阶段
* 修改老师给出的编译程序代码，使其可以执行，随后调试，实现读入pl0源程序进行编译产生中间代码，主要的修改在于因为老师给出的编译程序中一些符号不是ASCLL码，所以导致程序无法识别，所以要改为可以识别的符号，比如原文的小于等于号是不能被识别的，于是改为"<="，通过显示别小于号"<"，随后判断后面符号是否是"="来判断是否是小于等于比较操作，其他的修改类似。具体的代码如下（修改过的地方已加注释说明）：  
    ``` java  
        program  PL0(input, output);

        const
            norw = 11;    {保留字的个数}
            txmax = 100;  {标识符表长度}
            nmax = 14;    {数字的最大位数}
            al = 10;      {标识符的长度}
            amax = 2047;  {最大地址}
            levmax = 3;   {程序体嵌套的最大深度}
            cxmax = 200;  {代码数组的大小}

        type
            symbol = (nul, ident, number, plus, minus, times, slash, oddsym,
                eql, neq, lss, leq, gtr, geq, lparen, rparen, comma, semicolon,
                period, becomes, beginsym, endsym, ifsym, thensym,
                whilesym, dosym, callsym, constsym, varsym, procsym );

            alfa = packed array [1..al] of char;
            myObject = (constant, variable, myProcedure); {此处修改}
            symset = set of symbol;
            fct = (lit, opr, lod, sto, cal, int, jmp, jpc); {functions}
            instruction = packed record
                f : fct;  {功能码}
                l : 0..levmax; {相对层数}
                a : 0..amax; {相对地址}
                end;
        {
            LIT 0,a : 取常数a
            OPR 0,a : 执行运算a
            LOD l,a : 取层差为l的层﹑相对地址为a的变量
            STO l,a : 存到层差为l的层﹑相对地址为a的变量
            CAL l,a : 调用层差为l的过程
            INT 0,a : t寄存器增加a
            JMP 0,a : 转移到指令地址a处
            JPC 0,a : 条件转移到指令地址a处 
        }

        var
            ch : char; 		{最近读到的字符}
            sym : symbol; 	{最近读到的符号}
            id : alfa; 		{最近读到的标识符}
            num : integer; 	{最近读到的数}
            cc : integer; 	{当前行的字符计数}
            ll : integer; 	{当前行的长度}
            kk, err : integer;
            cx : integer; 	{代码数组的当前下标}
            line : array [1..81] of char;
            a : alfa;
            code : array [0..cxmax] of instruction;
            myWord : array [1..norw] of alfa; {此处修改}
            wsym : array [1..norw] of symbol;
            ssym : array [char] of symbol;
            mnemonic : array [fct] of packed array [1..5] of char;
            declbegsys, statbegsys, facbegsys : symset;
            table : array [0..txmax] of
                record
                    name : alfa;
                    case kind : myObject of constant : (val : integer);
                    variable, myProcedure : (level, adr : integer)
                end;
            {文件读写}
            iFile, oFile : string; {此处新增}
            iStream, oStream : text;

        procedure error (n : integer);
        begin 
            writeln(oStream, '****', ' ' : cc-1, '@', n : 2);  {此处修改}
            err := err + 1
        end {error};

        procedure getsym;

        var  i, j, k : integer;

        procedure getch;
        begin
            if cc = ll then
            begin
                if eof(iStream) then
                begin
                    write(oStream, 'PROGRAM INCOMPLETE'); 
                    halt    {此处修改}
                end;
                ll := 0; cc := 0; 
                write(oStream, cx : 5, ' ');
                while not eoln(iStream) do
                begin
                    ll := ll + 1; 
                    read(iStream, ch); 
                    write(oStream, ch);
                    line[ll] := upcase(ch)
                end;
                writeln(oStream); {此处修改}
                ll := ll + 1;
                read(iStream, line[ll]);
            end;
            cc := cc + 1; 
            ch := line[cc]
        end {getch};

        begin {getsym}
            while (ch = ' ') or (ord(ch) = 10) or (ord(ch) = 13)  do getch; {此处修改}
            if ch in ['A'..'Z'] then
            begin {标识符或保留字} 
                k := 0;
                repeat
                    if k < al then
                    begin 
                        k:= k + 1;
                        a[k] := ch
                    end;
                    getch
                until not (ch in ['A'..'Z', '0'..'9']); {此处修改原文非符号为not，下文的类似}

                if k >= kk  then kk := k {修改大于等于号}
                else
                    repeat 
                        a[kk] := ' '; 
                        kk := kk-1
                    until kk = k;

                id := a;  i := 1;  j := norw;
                repeat  k := (i+j) div 2;
                    if id <= myWord[k] then j := k-1; {修改word为与上文相应的标记myWord，同时小于等于号改为<=}
                    if id >= myWord[k] then i := k + 1 
                until i > j;

                if i-1 > j then sym := wsym[k] 
                else sym := ident
            end else
                if ch in ['0'..'9'] then
                begin {数字} 
                    k := 0;  num := 0;  sym := number;
                    repeat
                        num := 10*num + (ord(ch)-ord('0'));
                        k := k + 1;
                        getch;
                    until not (ch in ['0'..'9']);

                    if k > nmax then  error(30)
                end else if ch = ':' then{处理赋值号}
                begin  
                    getch;
                    if ch = '=' then
                    begin  
                        sym := becomes; 
                        getch
                    end
                    else  
                        sym := nul;
                end else if ch = '>' then {以下为新增内容，主要用于处理大于等于、小于等于、不等于}
                begin 
                    getch;
                    if ch = '=' then
                    begin
                        sym := geq;
                        getch
                    end
                    else
                        sym := gtr;
                end else if ch = '<' then{处理小于等于以及不等于}
                begin
                    getch;
                    if ch = '=' then
                    begin
                        sym := leq;
                        getch
                    end else if ch = '>' then
                    begin 
                        sym := neq;
                        getch
                    end else
                        sym := lss
                end else
                    begin  
                        sym := ssym[ch];
                        getch
                    end
        end {getsym};

        procedure gen(x : fct; y, z : integer);
        begin
            if cx > cxmax then 
            begin
                write(oStream, 'PROGRAM TOO LONG'); 
                halt
            end;
            with code[cx] do
            begin
                f := x;  l := y;  a := z
            end;
            cx := cx + 1
        end {gen};

        procedure test(s1, s2 : symset; n : integer);
        begin
            {测试，如果当前记号不属于集合S1,则报告错误n}
            if not (sym in s1) then
            begin  
                error(n);
                s1 := s1 + s2;
                while not (sym in s1) do getsym
            end
        end {test};

        procedure block(lev, tx : integer; fsys : symset);
        var
            dx : integer; 	{本过程数据空间分配下标}
            tx0 : integer; 	{本过程标识表起始下标}
            cx0 : integer; 	{本过程代码起始下标}


        procedure enter(k : myObject);
        begin {把myObject填入符号表中}
            tx := tx +1;
            with table[tx] do
            begin  
                name := id; kind := k;
                case k of
                    constant: 
                    begin
                        if num > amax then
                        begin 
                            error(30); num := 0 
                        end;
                        val := num
                    end;
                    variable: 
                    begin
                        level := lev;  adr := dx;  dx := dx +1;
                    end;
                    myProcedure : level := lev
                end
            end
        end {enter};

        {返回id在符号表的入口}
        function position(id : alfa) : integer;
        var  i : integer;
        begin {在标识符表中查标识符id}
            table[0].name := id; i := tx;
            while table[i].name <> id do i := i-1;
            position := i
        end {position};
        
        procedure constdeclaration;
        begin
            if sym = ident then
            begin 
                getsym;
                if sym in [eql, becomes] then {此处新增内容}
                begin
                    if sym = becomes then error(1);
                        getsym;
                    if sym = number then 
                    begin 
                        enter(constant); 
                        getsym
                    end
                    else
                        error(2)
                end else 
                    error(3)
            end else 
                error(4)
        end {constdeclaration};

        procedure vardeclaration;
        begin
            if sym = ident then
            begin  
                enter(variable);  
                getsym
            end else 
                error(4)
        end {vardeclaration};

        procedure listcode;
        var i : integer;
        begin  {打印本程序体生成的代码}
            for i := cx0 to cx-1 do
                with code[i] do
                    writeln(oStream, i:3, mnemonic[f]:6, l : 3, a : 5);
            
        end {listcode};

        procedure statement(fsys : symset);
        var  i, cx1, cx2 : integer;

        procedure  expression(fsys : symset);
        var  addop : symbol;

        procedure  term(fsys : symset);
        var  mulop : symbol;

        procedure  factor(fsys : symset);
        var i : integer;
        begin {factor}
            {测试当前的记号是否因子的开始符号, 否则出错, 跳过一些记号}
            test(facbegsys, fsys, 24);
            {如果当前的记号是否因子的开始符号}
            while sym in facbegsys do
            begin
                if sym = ident then
                begin
                    i := position(id);
                    if i = 0 then error(11) 
                    else
                        with table[i] do
                            case kind of
                                constant : gen(lit, 0, val);
                                variable : gen(lod, lev-level, adr);
                                myProcedure : error(21)
                            end;
                    getsym
                end else if sym = number then
                begin
                    if num > amax then
                    begin 
                        error(30);
                        num := 0 
                    end;
                    gen(lit, 0, num); 
                    getsym
                end else if sym = lparen then
                begin  
                    getsym;
                    expression([rparen]+fsys);
                    if sym = rparen then getsym
                    else error(22)
                end;
                test(fsys, [lparen], 23)
            end
        end {factor};
        
        begin {term}
            factor(fsys+[times, slash]);
            while sym in [times, slash] do
            begin
                mulop := sym;
                getsym;
                factor(fsys+[times, slash]);
                if mulop = times then gen(opr, 0, 4)
                else gen(opr, 0, 5)
            end
        end {term};

        begin {expression}
            if sym in [plus, minus] then
            begin 
                addop := sym;  
                getsym;
                term(fsys+[plus, minus]);
                if addop = minus then gen(opr, 0, 1)
            end else 
                term(fsys+[plus, minus]);
                while sym in [plus, minus] do
                begin
                    addop := sym;  
                    getsym;
                    term(fsys+[plus, minus]);
                    if addop = plus then gen(opr, 0, 2)
                    else gen(opr, 0, 3)
                end
        end {expression};

        procedure condition(fsys : symset);
        var  relop : symbol;
        begin{condition}
            if sym = oddsym then 	
            begin
                getsym;  
                expression(fsys);  
                gen(opr, 0, 6)
            end else
            begin
                expression([eql, neq, lss, gtr, leq, geq] + fsys);
                if not (sym in [eql, neq, lss, leq, gtr, geq]) then
                    error(20)  
                else
                begin
                    relop := sym; 
                    getsym;  
                    expression(fsys);
                    case relop of
                        eql : gen(opr, 0, 8);
                        neq : gen(opr, 0, 9);
                        lss : gen(opr, 0, 10);
                        geq : gen(opr, 0, 11);
                        gtr : gen(opr, 0, 12);
                        leq : gen(opr, 0, 13);
                    end
                end
            end
        end {condition};

        begin {statement}
            if sym = ident then 
            begin  
                i := position(id);
                if i = 0 then error(11) else
                if table[i].kind <> variable then {此处修改不等号为<>，下同}
                begin {对非变量赋值} 
                    error(12); 
                    i := 0; 
                end;
                getsym;
                if sym = becomes then getsym 
                else error(13);
                expression(fsys);
                if i <> 0 then 
                    with table[i] do gen(sto, lev-level, adr)
            end else if sym = callsym then
            begin  
                getsym;
                if sym <> ident then error(14) 
                else
                begin 
                    i := position(id);
                    if i = 0 then error(11) 
                    else
                        with table[i] do
                            if kind = myProcedure then 
                                gen(cal, lev-level, adr)
                            else error(15);
                    getsym
                end
            end else if sym = ifsym then
            begin
                getsym;  condition([thensym, dosym]+fsys);
                if sym = thensym then getsym 
                else error(16);
                cx1 := cx;  gen(jpc, 0, 0);
                statement(fsys);  
                code[cx1].a := cx
            end else if sym = beginsym then
            begin
                getsym;  
                statement([semicolon, endsym]+fsys);
                while sym in [semicolon]+statbegsys do
                begin
                    if sym = semicolon then getsym 
                    else error(10);
                    statement([semicolon, endsym]+fsys)
                end;
                if sym = endsym then getsym 
                else error(17)
            end else if sym = whilesym then
            begin
                cx1 := cx;  
                getsym;  
                condition([dosym]+fsys);
                cx2 := cx;  
                gen(jpc, 0, 0);
                if sym = dosym then getsym 
                else error(18);
                statement(fsys);  
                gen(jmp, 0, cx1);  
                code[cx2].a := cx
            end;
            test(fsys, [ ], 19)
        end {statement};

        begin {block}
            dx := 3;  
            tx0 := tx;  
            table[tx].adr := cx; 
            gen(jmp, 0, 0);
            if lev > levmax then error(32);
            repeat
                if sym = constsym then 
                begin  
                    getsym;
                    repeat 
                        constdeclaration;
                        while sym = comma do
                        begin 
                            getsym; 
                            constdeclaration 
                        end;
                        if sym = semicolon then
                        begin
                            getsym; 
                        end
                        else error(5)
                    until sym <> ident;
                end;
                if sym = varsym then
                begin  
                    getsym;
                    repeat 
                        vardeclaration;
                        while sym = comma do
                        begin  
                            getsym;  
                            vardeclaration  
                        end;
                        if sym = semicolon then getsym 
                        else error(5)
                    until sym <> ident;
                end;
                while sym = procsym do
                begin  getsym;
                    if sym = ident then
                    begin  
                        enter(myProcedure);  
                        getsym  
                    end else error(4);

                    if sym = semicolon then getsym 
                    else error(5);

                    block(lev+1, tx, [semicolon]+fsys);
                    if sym = semicolon then
                    begin getsym;
                        test(statbegsys+[ident, procsym], fsys, 6)
                    end
                    else error(5)
                end;
                {检测当前记号是否语句开始符号, 否则出错, 并跳过一些记号}
                test(statbegsys+[ident], declbegsys, 7)
            until not (sym in declbegsys);
            
            code[table[tx0].adr].a := cx;
            with table[tx0] do
            begin  
                adr := cx; {代码开始地址}
            end;
            cx0 := cx; gen(int, 0, dx);
            statement([semicolon, endsym]+fsys);
            gen(opr, 0, 0); {生成返回指令}
            test(fsys, [ ], 8);
            listcode;
        end  {block};

        procedure interpret;
        const stacksize = 500;
        var 
            p, b, t : integer;		{程序地址寄存器, 基地址寄存器,栈顶地址寄存器}
            i : instruction;		{指令寄存器}
            s : array [1..stacksize] of integer; {数据存储栈}

        function base(l : integer) : integer;
        var  b1 : integer;
        begin{base}
            b1 := b; {顺静态链求层差为l的层的基地址}
            while l > 0 do
            begin  
                b1 := s[b1];  
                l := l-1 
            end;
            base := b1
        end {base};

        begin{interpret}
            writeln(oStream, 'START PL/0');
            t := 0;  
            b := 1;  
            p := 0;
            s[1] := 0;  
            s[2] := 0;  
            s[3] := 0;
            repeat
                i := code[p];  
                p := p+1;
                with i do
                    case f of
                        lit : 
                        begin
                            t := t+1;  
                            s[t] := a
                        end;
                        opr:
                            case a of {运算}
                                0 : begin {返回}
                                        t := b-1;  p := s[t+3];  b := s[t+2];
                                    end;
                                1 : s[t] := -s[t];
                                2 : begin
                                        t := t-1;  s[t] := s[t] + s[t+1]
                                    end;
                                3 : begin
                                        t := t-1;  s[t] := s[t]-s[t+1]
                                    end;
                                4 : begin
                                        t := t-1;  s[t] := s[t] * s[t+1]
                                    end;
                                5 : begin
                                        t := t-1;  s[t] := s[t] div s[t+1]
                                    end;
                                6 : s[t] := ord(odd(s[t]));
                                8 : begin  t := t-1;
                                        s[t] := ord(s[t] = s[t+1])
                                    end;
                                9 : begin  t := t-1;
                                        s[t] := ord(s[t] <> s[t+1])
                                    end;
                                10 : begin  t := t-1;
                                        s[t] := ord(s[t] < s[t+1])
                                    end;
                                11 : begin  t := t-1;
                                        s[t] := ord(s[t] >= s[t+1])
                                    end;
                                12 : begin  t := t-1;
                                        s[t] := ord(s[t] > s[t+1])
                                    end;
                                13 : begin  t := t-1;
                                    s[t] := ord(s[t] <= s[t+1])
                                    end;
                            end;
                        lod : 
                        begin
                            t := t + 1;  s[t] := s[base(l) + a]
                        end;
                        sto : 
                        begin
                            s[base(l) + a] := s[t];  
                            writeln(oStream, s[t]);
                            t := t-1
                        end;
                        cal: 
                        begin {generate new block mark}
                            s[t+1] := base( l );  s[t+2] := b;
                            s[t+3] := p;
                            b := t+1;  p := a
                        end;
                        int : t := t + a;
                        jmp : p := a;
                        jpc : 
                        begin
                            if s[t] = 0 then p := a;
                            t := t-1
                        end
                    end {with, case}
            until p = 0;
            write(oStream, 'END PL/0');
        end {interpret};

        begin  {主程序}

            {read} 
            writeln('Input your pl0 source file:');
            readln(iFile);
            Assign(iStream, iFile);
            Reset(iStream);

            {write}
            writeln('Write your result into a file you want:');
            readln(oFile);
            Assign(oStream, oFile);
            Rewrite(oStream);

            {初始化}
            for ch := 'A' to ';' do ssym[ch] := nul;

            {保留字}
            myWord[1] := 'BEGIN     '; myWord[2] := 'CALL      ';
            myWord[3] := 'CONST     '; myWord[4] := 'DO        ';
            myWord[5] := 'END       '; myWord[6] := 'IF        ';
            myWord[7] := 'ODD       '; myWord[8] := 'PROCEDURE ';
            myWord[9] := 'THEN      '; myWord[10] := 'VAR       ';
            myWord[11] := 'WHILE     ';

            {保留字的记号}
            wsym[1] := beginsym;	wsym[2] := callsym;
            wsym[3] := constsym;	wsym[4] := dosym;
            wsym[5] := endsym;		wsym[6] := ifsym;
            wsym[7] := oddsym;		wsym[8] := procsym;
            wsym[9] := thensym; 	wsym[10] := varsym;
            wsym[11] := whilesym;

            {以下删掉原本的小于等于，因为不是ASCLL码无法识别，在上文使用程序判断语句代替}
            ssym['+'] := plus;		ssym['-'] := minus;
            ssym['*'] := times;		ssym['/'] := slash;
            ssym['('] := lparen;	ssym[')'] := rparen;
            ssym['='] := eql;		ssym[','] := comma;
            ssym['.'] := period;	ssym['<'] := lss;       
            ssym['>'] := gtr;		ssym[';'] := semicolon;

            {算符和标点符号的记号}
            mnemonic[lit] := 'LIT  ';		mnemonic[opr] := 'OPR  ';
            mnemonic[lod] := 'LOD  ';		mnemonic[sto] := 'STO  ';
            mnemonic[cal] := 'CAL  ';		mnemonic[int] := 'INT  ';
            mnemonic[jmp] := 'JMP  ';		mnemonic[jpc] := 'JPC  ';
            
            {中间代码指令的字符串}
            declbegsys := [constsym, varsym, procsym];
            {说明语句的开始符号}
            statbegsys := [beginsym, callsym, ifsym, whilesym];
            {语句的开始符号}
            facbegsys := [ident, number, lparen];
            {因子的开始符号}
            {page(output); }
            {发现错误的个数}
            err := 0; 
            cc := 0; cx := 0; ll := 0; ch := ' '; kk := al;  
            getsym;
            {处理程序体}
            block(0, 0, [period] + declbegsys + statbegsys);

            if sym <> period then error(9);
            if err = 0 then interpret
            else write(oStream, 'ERRORS IN PL/0 PROGRAM');

            {关闭输入输出流}
            close(iStream);
            close(oStream);
        end.
    ```  
* 用于编译的pl0源程序：  
    ```  
        const m = 7, n = 85;
        var  x, y, z, q, r;

        procedure  multiply;
        var  a, b;
        begin  
            a := x;  b := y;  z := 0;
            while b > 0 do
            begin  
                if odd b then z := z + a;
                a := 2*a ;  
                b := b/2 ;
            end
        end;

        procedure  divide;
        var  w;
        begin  
            r := x;  q := 0;  w := y;
            while w <= r do w := 2*w ;
            while w > y do
            begin  
                q := 2*q;  w := w/2;
                if w <= r then
                begin  
                    r := r-w;  
                    q := q+1 
                end
            end
        end;

        procedure  gcd;
        var f, g;
        begin  
            f := x;  
            g := y;
            while f <> g do
            begin
                if f < g then g := g-f;
                if g < f then f := f-g;
            end;
            z := f
        end;

        begin 
            x := m; 
            y := n; 
            call multiply;
            x := 25;
            y := 3; 
            call divide;
            x := 84;
            y := 36;
            call gcd;
        end.
    ```  
* 对给定的pl0源程序进行编译后，得到的中间代码如下：  
```  
    2 INT    0    5
    3 LOD    1    3
    4 STO    0    3
    5 LOD    1    4
    6 STO    0    4
    7 LIT    0    0
    8 STO    1    5
    9 LOD    0    4
    10 LIT    0    0
    11 OPR    0   12
    12 JPC    0   29
    13 LOD    0    4
    14 OPR    0    6
    15 JPC    0   20
    16 LOD    1    5
    17 LOD    0    3
    18 OPR    0    2
    19 STO    1    5
    20 LIT    0    2
    21 LOD    0    3
    22 OPR    0    4
    23 STO    0    3
    24 LOD    0    4
    25 LIT    0    2
    26 OPR    0    5
    27 STO    0    4
    28 JMP    0    9
    29 OPR    0    0
    31 INT    0    4
    32 LOD    1    3
    33 STO    1    7
    34 LIT    0    0
    35 STO    1    6
    36 LOD    1    4
    37 STO    0    3
    38 LOD    0    3
    39 LOD    1    7
    40 OPR    0   13
    41 JPC    0   47
    42 LIT    0    2
    43 LOD    0    3
    44 OPR    0    4
    45 STO    0    3
    46 JMP    0   38
    47 LOD    0    3
    48 LOD    1    4
    49 OPR    0   12
    50 JPC    0   72
    51 LIT    0    2
    52 LOD    1    6
    53 OPR    0    4
    54 STO    1    6
    55 LOD    0    3
    56 LIT    0    2
    57 OPR    0    5
    58 STO    0    3
    59 LOD    0    3
    60 LOD    1    7
    61 OPR    0   13
    62 JPC    0   71
    63 LOD    1    7
    64 LOD    0    3
    65 OPR    0    3
    66 STO    1    7
    67 LOD    1    6
    68 LIT    0    1
    69 OPR    0    2
    70 STO    1    6
    71 JMP    0   47
    72 OPR    0    0
    74 INT    0    5
    75 LOD    1    3
    76 STO    0    3
    77 LOD    1    4
    78 STO    0    4
    79 LOD    0    3
    80 LOD    0    4
    81 OPR    0    9
    82 JPC    0  100
    83 LOD    0    3
    84 LOD    0    4
    85 OPR    0   10
    86 JPC    0   91
    87 LOD    0    4
    88 LOD    0    3
    89 OPR    0    3
    90 STO    0    4
    91 LOD    0    4
    92 LOD    0    3
    93 OPR    0   10
    94 JPC    0   99
    95 LOD    0    3
    96 LOD    0    4
    97 OPR    0    3
    98 STO    0    3
    99 JMP    0   79
    100 LOD    0    3
    101 STO    1    5
    102 OPR    0    0
    103 INT    0    8
    104 LIT    0    7
    105 STO    0    3
    106 LIT    0   85
    107 STO    0    4
    108 CAL    0    2
    109 LIT    0   25
    110 STO    0    3
    111 LIT    0    3
    112 STO    0    4
    113 CAL    0   31
    114 LIT    0   84
    115 STO    0    3
    116 LIT    0   36
    117 STO    0    4
    118 CAL    0   74
    119 OPR    0    0
```
* 对给定的pl0源程序进行编译后，得到的栈中数据如下：  
```
    START PL/0
    7
    85
    7
    85
    0
    7
    14
    42
    28
    21
    35
    56
    10
    112
    5
    147
    224
    2
    448
    1
    595
    896
    0
    25
    3
    25
    0
    3
    6
    12
    24
    48
    0
    24
    1
    1
    2
    12
    4
    6
    8
    3
    84
    36
    84
    36
    48
    12
    24
    12
    12
    END PL/0
```

### 第二阶段
* 此阶段我选择使用文件读写，修改给出的pl0源程序，在里面增加read和write语句来进行测试。
* 在第一阶段的基础上在编译程序中增加read、write语句，增加READ和WRITE保留字以及增加相关操作来实现键盘读写功能，具体代码如下（新增内容已加注释进行标注）：  
    ```  
        program  PL0(input, output);

        const
            norw = 13;    {保留字的个数, 比阶段1新增了2个}
            txmax = 100;  {标识符表长度}
            nmax = 14;    {数字的最大位数}
            al = 10;      {标识符的长度}
            amax = 2047;  {最大地址}
            levmax = 3;   {程序体嵌套的最大深度}
            cxmax = 200;  {代码数组的大小}

        type
            symbol = (nul, ident, number, plus, minus, times, slash, oddsym, readsym, writesym,
                eql, neq, lss, leq, gtr, geq, lparen, rparen, comma, semicolon,
                period, becomes, beginsym, endsym, ifsym, thensym,
                whilesym, dosym, callsym, constsym, varsym, procsym ); {新增读写两个标识}

            alfa = packed array [1..al] of char;
            myObject = (constant, variable, myProcedure); {此处修改}
            symset = set of symbol;
            fct = (lit, opr, lod, sto, cal, int, jmp, jpc, pRead, pWrite); {functions}
            instruction = packed record
                f : fct;  		{功能码}
                l : 0..levmax; 	{相对层数}
                a : 0..amax; 	{相对地址}
                end;
        {
            LIT 0,a : 取常数a
            OPR 0,a : 执行运算a
            LOD l,a : 取层差为l的层﹑相对地址为a的变量
            STO l,a : 存到层差为l的层﹑相对地址为a的变量
            CAL l,a : 调用层差为l的过程
            INT 0,a : t寄存器增加a
            JMP 0,a : 转移到指令地址a处
            JPC 0,a : 条件转移到指令地址a处
        }

        var
            ch : char; 		{最近读到的字符}
            sym : symbol; 	{最近读到的符号}
            id : alfa; 		{最近读到的标识符}
            num : integer; 	{最近读到的数}
            cc : integer; 	{当前行的字符计数}
            ll : integer; 	{当前行的长度}
            kk, err : integer;
            cx : integer; 	{代码数组的当前下标}
            line : array [1..81] of char;
            a : alfa;
            code : array [0..cxmax] of instruction;
            myWord : array [1..norw] of alfa; {此处修改}
            wsym : array [1..norw] of symbol;
            ssym : array [char] of symbol;
            mnemonic : array [fct] of packed array [1..5] of char;
            declbegsys, statbegsys, facbegsys : symset;
            table : array [0..txmax] of
                record
                    name : alfa;
                    case kind : myObject of constant : (val : integer);
                    variable, myProcedure : (level, adr : integer)
                end;
            {文件读写}
            iFile, oFile : string; {此处新增}
            iStream, oStream : text;
            {
                用Read(文件变量,变量列表);语句读取指定的文件。
                用Write(文件变量,变量列表);语句写入指定的文件。
            }

        procedure error (n : integer);
        begin 
            writeln(oStream, '****', ' ' : cc-1, '@', n : 2); {此处修改}
            err := err + 1
        end {error};

        procedure getsym;

        var  i, j, k : integer;

        procedure getch;
        begin
            if cc = ll then
            begin
                if eof(iStream) then
                begin
                    write(oStream, 'PROGRAM INCOMPLETE'); 
                    halt    {此处修改}
                end;
                ll := 0; cc := 0; 
                write(oStream, cx : 5, ' ');
                while not eoln(iStream) do
                begin
                    ll := ll + 1; 
                    read(iStream, ch); 
                    write(oStream, ch);
                    line[ll] := upcase(ch)
                end;
                writeln(oStream); {此处修改}
                ll := ll + 1;
                read(iStream, line[ll]);
            end;
            cc := cc + 1; 
            ch := line[cc]
        end {getch};

        begin {getsym}
            while (ch = ' ') or (ord(ch) = 10) or (ord(ch) = 13)  do getch;
            if ch in ['A'..'Z'] then
            begin {标识符或保留字} 
                k := 0;
                repeat
                    if k < al then
                    begin 
                        k:= k + 1;
                        a[k] := ch
                    end;
                    getch
                until not (ch in ['A'..'Z', '0'..'9']); {此处修改原文非符号为not，下文的类似}

                if k >= kk  then kk := k {修改大于等于号}
                else
                    repeat 
                        a[kk] := ' '; 
                        kk := kk-1
                    until kk = k;
                id := a;  i := 1;  j := norw;
                repeat  k := (i+j) div 2;
                    if id <= myWord[k] then j := k-1; {修改word为与上文相应的标记myWord，同时小于等于号改为<=}
                    if id >= myWord[k] then i := k + 1
                until i > j;
                if i-1 > j then sym := wsym[k] 
                else sym := ident
            end else
                if ch in ['0'..'9'] then
                begin {数字} 
                    k := 0;  num := 0;  sym := number;
                    repeat
                        num := 10*num + (ord(ch)-ord('0'));
                        k := k + 1;
                        getch;
                    until not (ch in ['0'..'9']);

                    if k > nmax then  error(30)
                end else if ch = ':' then {以下为新增内容，主要用于处理大于等于、小于等于、不等于}
                begin  
                    getch;
                    if ch = '=' then
                    begin  
                        sym := becomes; 
                        getch
                    end
                    else  
                        sym := nul;
                end else if ch = '>' then{处理大于等于}
                begin 
                    getch;
                    if ch = '=' then
                    begin
                        sym := geq;
                        getch
                    end
                    else
                        sym := gtr;
                end else if ch = '<' then{处理小于等于以及不等于}
                begin
                    getch;
                    if ch = '=' then
                    begin
                        sym := leq;
                        getch
                    end else if ch = '>' then
                    begin 
                        sym := neq;
                        getch
                    end else
                        sym := lss
                end else
                    begin  
                        sym := ssym[ch];
                        getch
                    end
        end {getsym};

        procedure gen(x : fct; y, z : integer);
        begin
            if cx > cxmax then 
            begin
                write(oStream, 'PROGRAM TOO LONG'); 
                halt
            end;
            with code[cx] do 
            begin
                f := x;  l := y;  a := z
            end;
            cx := cx + 1
        end {gen};

        procedure test(s1, s2 : symset; n : integer);
        begin
            {测试，如果当前记号不属于集合S1,则报告错误n}
            if not (sym in s1) then
            begin  
                error(n);
                s1 := s1 + s2;
                while not (sym in s1) do getsym
            end
        end {test};

        procedure block(lev, tx : integer; fsys : symset);
        var
            dx : integer; 	{本过程数据空间分配下标}
            tx0 : integer; 	{本过程标识表起始下标}
            cx0 : integer; 	{本过程代码起始下标}


        procedure enter(k : myObject);
        begin {把myObject填入符号表中}
            tx := tx +1;
            with table[tx] do
            begin  
                name := id; kind := k;
                case k of
                    constant: 
                    begin
                        if num > amax then
                        begin 
                            error(30); num := 0 
                        end;
                        val := num
                    end;
                    variable: 
                    begin
                        level := lev;  adr := dx;  dx := dx +1;
                    end;
                    myProcedure : level := lev
                end
            end
        end {enter};

        {返回id在符号表的入口}
        function position(id : alfa) : integer;
        var  i : integer;
        begin {在标识符表中查标识符id}
            table[0].name := id; i := tx;
            while table[i].name <> id do i := i-1;
            position := i
        end {position};
        
        procedure constdeclaration;
        begin
            if sym = ident then
            begin 
                getsym;
                if sym in [eql, becomes] then {此处新增内容}
                begin
                    if sym = becomes then error(1);
                        getsym;
                    if sym = number then 
                    begin 
                        enter(constant); 
                        getsym
                    end
                    else
                        error(2)
                end else 
                    error(3)
            end else 
                error(4)
        end {constdeclaration};

        procedure vardeclaration;
        begin
            if sym = ident then
            begin  
                enter(variable);  
                getsym
            end else 
                error(4)
        end {vardeclaration};

        procedure listcode;
        var i : integer;
        begin  {打印本程序体生成的代码}
            for i := cx0 to cx-1 do
                with code[i] do
                    writeln(oStream, i:3, mnemonic[f]:6, l : 3, a : 5);
            
        end {listcode};

        procedure statement(fsys : symset);
        var  i, cx1, cx2 : integer;

        procedure  expression(fsys : symset);
        var  addop : symbol;

        procedure  term(fsys : symset);
        var  mulop : symbol;

        procedure  factor(fsys : symset);
        var i : integer;
        begin {factor}
            {测试当前的记号是否因子的开始符号, 否则出错, 跳过一些记号}
            test(facbegsys, fsys, 24);
            {如果当前的记号是否因子的开始符号}
            while sym in facbegsys do
            begin
                if sym = ident then
                begin
                    i := position(id);
                    if i = 0 then error(11) 
                    else
                        with table[i] do
                            case kind of
                                constant : gen(lit, 0, val);
                                variable : gen(lod, lev-level, adr);
                                myProcedure : error(21)
                            end;
                    getsym
                end else if sym = number then
                begin
                    if num > amax then
                    begin 
                        error(30);
                        num := 0 
                    end;
                    gen(lit, 0, num); 
                    getsym
                end else if sym = lparen then
                begin 
                    getsym;
                    expression([rparen]+fsys);
                    if sym = rparen then getsym
                    else error(22)
                end;
                test(fsys, [lparen], 23)
            end
        end {factor};
        
        begin {term}
            factor(fsys+[times, slash]);
            while sym in [times, slash] do
            begin
                mulop := sym;
                getsym;
                factor(fsys+[times, slash]);
                if mulop = times then gen(opr, 0, 4)
                else gen(opr, 0, 5)
            end
        end {term};

        begin {expression}
            if sym in [plus, minus] then
            begin 
                addop := sym;  
                getsym;
                term(fsys+[plus, minus]);
                if addop = minus then gen(opr, 0, 1)
            end else 
                term(fsys+[plus, minus]);
                while sym in [plus, minus] do
                begin
                    addop := sym;  
                    getsym;
                    term(fsys+[plus, minus]);
                    if addop = plus then gen(opr, 0, 2)
                    else gen(opr, 0, 3)
                end
        end {expression};

        procedure condition(fsys : symset);
        var  relop : symbol;
        begin{condition}
            if sym = oddsym then 	
            begin
                getsym;  
                expression(fsys);  
                gen(opr, 0, 6)
            end else
            begin
                expression([eql, neq, lss, gtr, leq, geq] + fsys);
                if not (sym in [eql, neq, lss, leq, gtr, geq]) then
                    error(20)  
                else
                begin
                    relop := sym; 
                    getsym;  
                    expression(fsys);
                    case relop of
                        eql : gen(opr, 0, 8);
                        neq : gen(opr, 0, 9);
                        lss : gen(opr, 0, 10);
                        geq : gen(opr, 0, 11);
                        gtr : gen(opr, 0, 12);
                        leq : gen(opr, 0, 13);
                    end
                end
            end
        end {condition};

        begin {statement}
            if sym = ident then 
            begin  
                i := position(id);
                if i = 0 then error(11) else
                if table[i].kind <> variable then {此处修改不等号为<>，下同}
                begin {对非变量赋值} 
                    error(12); 
                    i := 0; 
                end;
                getsym;
                if sym = becomes then getsym 
                else error(13);
                expression(fsys);
                if i <> 0 then
                    with table[i] do gen(sto, lev-level, adr)
            end else if sym = callsym then
            begin  
                getsym;
                if sym <> ident then error(14) 
                else
                begin 
                    i := position(id);
                    if i = 0 then error(11) 
                    else
                        with table[i] do
                            if kind = myProcedure then 
                                gen(cal, lev-level, adr)
                            else error(15);
                    getsym
                end
            end else if sym = ifsym then
            begin
                getsym;  condition([thensym, dosym]+fsys);
                if sym = thensym then getsym 
                else error(16);
                cx1 := cx;  gen(jpc, 0, 0);
                statement(fsys);  
                code[cx1].a := cx
            end else if sym = beginsym then
            begin
                getsym;  
                statement([semicolon, endsym]+fsys);
                while sym in [semicolon]+statbegsys do
                begin
                    if sym = semicolon then getsym 
                    else error(10);
                    statement([semicolon, endsym]+fsys)
                end;
                if sym = endsym then getsym 
                else error(17)
            end else if sym = whilesym then
            begin
                cx1 := cx;  
                getsym;  
                condition([dosym]+fsys);
                cx2 := cx;  
                gen(jpc, 0, 0);
                if sym = dosym then getsym 
                else error(18);
                statement(fsys);  
                gen(jmp, 0, cx1);  
                code[cx2].a := cx
            end else if sym = readsym then {新增读功能}
            begin
                getsym;
                if sym = lparen then
                begin  
                    getsym;
                    repeat 
                        if sym = ident then 
                        begin  
                            i := position(id); 
                            if i = 0 then error(11) else
                            if table[i].kind <> variable then
                            begin 
                                error(12); 
                                i := 0; 
                            end else 
                                with table[i] do gen(pRead, lev-level, adr);
                            getsym;
                            if sym = comma then getsym;
                        end else 
                            error(12);
                    until sym <> ident; 
                    if sym = rparen then getsym else error(22);
                end else
                    error(33);
            end else if sym = writesym then {新增写功能}
            begin
                getsym;
                if sym = lparen then 
                begin  
                    getsym;
                    repeat 
                        if sym = ident then 
                        begin  
                            i := position(id); 
                            if i = 0 then error(11) else
                            if table[i].kind <> variable then
                            begin 
                                error(12); 
                                i := 0; 
                            end else 
                                with table[i] do gen(pWrite, lev-level, adr);
                            getsym;
                            if sym = comma then getsym;
                        end else 
                            error(12);
                    until sym <> ident; 
                    if sym = rparen then getsym else error(22);
                end else
                    error(33);
            end; 
            test(fsys, [ ], 19)
        end {statement};

        begin {block}
            dx := 3;  
            tx0 := tx;  
            table[tx].adr := cx; 
            gen(jmp, 0, 0);
            if lev > levmax then error(32);
            repeat
                if sym = constsym then 
                begin  
                    getsym;
                    repeat 
                        constdeclaration;
                        while sym = comma do
                        begin 
                            getsym; 
                            constdeclaration 
                        end;
                        if sym = semicolon then
                        begin
                            getsym; 
                        end
                        else error(5)
                    until sym <> ident;
                end;
                if sym = varsym then
                begin  
                    getsym;
                    repeat 
                        vardeclaration;
                        while sym = comma do
                        begin  
                            getsym;  
                            vardeclaration  
                        end;
                        if sym = semicolon then getsym 
                        else error(5)
                    until sym <> ident;
                end;
                while sym = procsym do
                begin  getsym;
                    if sym = ident then
                    begin  
                        enter(myProcedure);  
                        getsym  
                    end else error(4);

                    if sym = semicolon then getsym 
                    else error(5);

                    block(lev+1, tx, [semicolon]+fsys);
                    if sym = semicolon then
                    begin getsym;
                        test(statbegsys+[ident, procsym], fsys, 6)
                    end
                    else error(5)
                end;
                {检测当前记号是否语句开始符号, 否则出错, 并跳过一些记号}
                test(statbegsys+[ident], declbegsys, 7)
            until not (sym in declbegsys);
            
            code[table[tx0].adr].a := cx;
            with table[tx0] do
            begin  
                adr := cx; {代码开始地址}
            end;
            cx0 := cx; gen(int, 0, dx);
            statement([semicolon, endsym]+fsys);
            gen(opr, 0, 0); {生成返回指令}
            test(fsys, [ ], 8);
            listcode;
        end  {block};

        procedure interpret;
        const stacksize = 500;
        var 
            p, b, t : integer;		{程序地址寄存器, 基地址寄存器,栈顶地址寄存器}
            i : instruction;		{指令寄存器}
            s : array [1..stacksize] of integer; {数据存储栈}

        function base(l : integer) : integer;
        var  b1 : integer;
        begin{base}
            b1 := b; {顺静态链求层差为l的层的基地址}
            while l > 0 do
            begin  
                b1 := s[b1];  
                l := l-1 
            end;
            base := b1
        end {base};

        begin{interpret}
            writeln(oStream, 'START PL/0');
            t := 0;  
            b := 1;  
            p := 0;
            s[1] := 0;  
            s[2] := 0;  
            s[3] := 0;
            repeat
                i := code[p];  
                p := p+1;
                with i do
                    case f of
                        lit : 
                        begin
                            t := t+1;  
                            s[t] := a
                        end;
                        opr:
                            case a of {运算}
                                0 : begin {返回}
                                        t := b-1;  p := s[t+3];  b := s[t+2];
                                    end;
                                1 : s[t] := -s[t];
                                2 : begin
                                        t := t-1;  s[t] := s[t] + s[t+1]
                                    end;
                                3 : begin
                                        t := t-1;  s[t] := s[t]-s[t+1]
                                    end;
                                4 : begin
                                        t := t-1;  s[t] := s[t] * s[t+1]
                                    end;
                                5 : begin
                                        t := t-1;  s[t] := s[t] div s[t+1]
                                    end;
                                6 : s[t] := ord(odd(s[t]));
                                8 : begin  t := t-1;
                                        s[t] := ord(s[t] = s[t+1])
                                    end;
                                9 : begin  t := t-1;
                                        s[t] := ord(s[t] <> s[t+1])
                                    end;
                                10 : begin  t := t-1;
                                        s[t] := ord(s[t] < s[t+1])
                                    end;
                                11 : begin  t := t-1;
                                        s[t] := ord(s[t] >= s[t+1])
                                    end;
                                12 : begin  t := t-1;
                                        s[t] := ord(s[t] > s[t+1])
                                    end;
                                13 : begin  t := t-1;
                                    s[t] := ord(s[t] <= s[t+1])
                                    end;
                            end;
                        lod : 
                        begin
                            t := t + 1;  s[t] := s[base(l) + a]
                        end;
                        sto : 
                        begin
                            s[base(l) + a] := s[t];  
                            writeln(oStream, s[t]);
                            t := t-1
                        end;
                        cal: 
                        begin {generate new block mark}
                            s[t+1] := base( l );  s[t+2] := b;
                            s[t+3] := p;
                            b := t+1;  p := a
                        end;
                        int : t := t + a;
                        jmp : p := a;
                        jpc : 
                        begin
                            if s[t] = 0 then p := a;
                            t := t-1
                        end;
                        pRead :
                        begin
                            read(s[base(l) + a]);
                            t := t+1
                        end;
                        pWrite :
                        begin
                            writeln(s[base(l) + a]);
                            t := t + 1
                        end;
                    end {with, case}
            until p = 0;
            write(oStream, 'END PL/0');
        end {interpret};

        begin  {主程序}

            {read} 
            writeln('Input your pl0 source file:');
            readln(iFile);
            Assign(iStream, iFile);
            Reset(iStream);

            {write}
            writeln('Write your result into a file you want:');
            readln(oFile);
            Assign(oStream, oFile);
            Rewrite(oStream);

            {初始化}
            for ch := 'A' to ';' do ssym[ch] := nul;

            {保留字，新增read、write}
            myWord[1] := 'BEGIN     '; myWord[2] := 'CALL      ';
            myWord[3] := 'CONST     '; myWord[4] := 'DO        ';
            myWord[5] := 'END       '; myWord[6] := 'IF        ';
            myWord[7] := 'ODD       '; myWord[8] := 'PROCEDURE ';
            myWord[9] := 'READ      ';
            myWord[10] := 'THEN      '; 
            myWord[11] := 'VAR       ';
            myWord[12] := 'WHILE     ';
            myWord[13] := 'WRITE     ';

            {保留字的记号}
            wsym[1] := beginsym;	wsym[2] := callsym;
            wsym[3] := constsym;	wsym[4] := dosym;
            wsym[5] := endsym;		wsym[6] := ifsym;
            wsym[7] := oddsym;		wsym[8] := procsym;
            wsym[9] := readsym;     wsym[10] := thensym; 	
            wsym[11] := varsym;		wsym[12] := whilesym;   
            wsym[13] := writesym;
            
            {以下删掉原本的小于等于，因为不是ASCLL码无法识别，在上文使用程序判断语句代替}
            ssym['+'] := plus;		ssym['-'] := minus;
            ssym['*'] := times;		ssym['/'] := slash;
            ssym['('] := lparen;	ssym[')'] := rparen;
            ssym['='] := eql;		ssym[','] := comma;
            ssym['.'] := period;	ssym['<'] := lss;       
            ssym['>'] := gtr;		ssym[';'] := semicolon;

            {算符和标点符号的记号}
            mnemonic[lit] := 'LIT  ';		mnemonic[opr] := 'OPR  ';
            mnemonic[lod] := 'LOD  ';		mnemonic[sto] := 'STO  ';
            mnemonic[cal] := 'CAL  ';		mnemonic[int] := 'INT  ';
            mnemonic[jmp] := 'JMP  ';		mnemonic[jpc] := 'JPC  ';
            mnemonic[pRead] := 'READ ';
            mnemonic[pWrite] := 'WRITE';
            
            {中间代码指令的字符串}
            declbegsys := [constsym, varsym, procsym];
            {说明语句的开始符号}
            statbegsys := [beginsym, callsym, ifsym, whilesym, readsym, writesym];
            {语句的开始符号}
            facbegsys := [ident, number, lparen];
            {因子的开始符号}
            {page(output); }
            {发现错误的个数}
            err := 0; 
            cc := 0; cx := 0; ll := 0; ch := ' '; kk := al;  
            getsym;
            {处理程序体}
            block(0, 0, [period] + declbegsys + statbegsys);

            if sym <> period then error(9);
            if err = 0 then interpret
            else write(oStream, 'ERRORS IN PL/0 PROGRAM');

            {关闭输入输出流}
            close(iStream);
            close(oStream);
        end.
    ```  
* 修改后的pl0源程序如下（新增read和write语句）： 
    ```  
        const m = 7, n = 85;

        var x, y, z, r, q, p;

        procedure  multiply;
            var  a, b;
            begin  
            a := x;  
            b := y;  
            z := 0;
            while b > 0 do
            begin  
                if odd b then z := z + a;
                a := 2*a ;  
                b := b/2 ;
            end
        end;

        procedure  divide;
            var  w;
            begin  
            r := x;  
            q := 0;  
            w := y;
            while w <= r do w := 2*w;
            while w > y do
            begin  
                q := 2*q;  
                w := w/2;
                if w <= r then
                begin  
                    r := r-w;  
                    q := q+1 
                end
            end
        end;

        procedure gcd;
            var f, g;
            begin  
            f := x;  
            g := y;
            while f <> g do
            begin
                if f < g then g := g-f;
                if g < f then f := f-g;
            end;
            z := f
        end;

        begin 
            read(x,y);  
            call multiply;
            p := z;
            write(p);
            call divide;
            write(q);
            call gcd;
            write(z)
        end.
    ```   
* 运行编译程序对pl0源程序进行编译，得到的中间代码如下：    
```  
    2 INT    0    5
    3 LOD    1    3
    4 STO    0    3
    5 LOD    1    4
    6 STO    0    4
    7 LIT    0    0
    8 STO    1    5
    9 LOD    0    4
    10 LIT    0    0
    11 OPR    0   12
    12 JPC    0   29
    13 LOD    0    4
    14 OPR    0    6
    15 JPC    0   20
    16 LOD    1    5
    17 LOD    0    3
    18 OPR    0    2
    19 STO    1    5
    20 LIT    0    2
    21 LOD    0    3
    22 OPR    0    4
    23 STO    0    3
    24 LOD    0    4
    25 LIT    0    2
    26 OPR    0    5
    27 STO    0    4
    28 JMP    0    9
    29 OPR    0    0
    31 INT    0    4
    32 LOD    1    3
    33 STO    1    6
    34 LIT    0    0
    35 STO    1    7
    36 LOD    1    4
    37 STO    0    3
    38 LOD    0    3
    39 LOD    1    6
    40 OPR    0   13
    41 JPC    0   47
    42 LIT    0    2
    43 LOD    0    3
    44 OPR    0    4
    45 STO    0    3
    46 JMP    0   38
    47 LOD    0    3
    48 LOD    1    4
    49 OPR    0   12
    50 JPC    0   72
    51 LIT    0    2
    52 LOD    1    7
    53 OPR    0    4
    54 STO    1    7
    55 LOD    0    3
    56 LIT    0    2
    57 OPR    0    5
    58 STO    0    3
    59 LOD    0    3
    60 LOD    1    6
    61 OPR    0   13
    62 JPC    0   71
    63 LOD    1    6
    64 LOD    0    3
    65 OPR    0    3
    66 STO    1    6
    67 LOD    1    7
    68 LIT    0    1
    69 OPR    0    2
    70 STO    1    7
    71 JMP    0   47
    72 OPR    0    0
    74 INT    0    5
    75 LOD    1    3
    76 STO    0    3
    77 LOD    1    4
    78 STO    0    4
    79 LOD    0    3
    80 LOD    0    4
    81 OPR    0    9
    82 JPC    0  100
    83 LOD    0    3
    84 LOD    0    4
    85 OPR    0   10
    86 JPC    0   91
    87 LOD    0    4
    88 LOD    0    3
    89 OPR    0    3
    90 STO    0    4
    91 LOD    0    4
    92 LOD    0    3
    93 OPR    0   10
    94 JPC    0   99
    95 LOD    0    3
    96 LOD    0    4
    97 OPR    0    3
    98 STO    0    3
    99 JMP    0   79
    100 LOD    0    3
    101 STO    1    5
    102 OPR    0    0
    103 INT    0    9
    104 READ   0    3
    105 READ   0    4
    106 CAL    0    2
    107 LOD    0    5
    108 STO    0    8
    109 WRITE  0    8
    110 CAL    0   31
    111 WRITE  0    7
    112 CAL    0   74
    113 WRITE  0    5
    114 OPR    0    0
```  
* 运行编译程序对pl0源程序进行编译运行，输入如下数据： 
```  
    30
    5
```  
* 输出数据如下，分别为上述输入数据相乘的结果、相除的结果和它们的最大公约数：  
```  
    150
    6
    5
```  
* 整个栈的数据如下：
```  
    START PL/0
    30
    5
    0
    30
    60
    2
    120
    1
    150
    240
    0
    150
    30
    0
    5
    10
    20
    40
    0
    20
    10
    1
    2
    10
    0
    3
    6
    5
    30
    5
    25
    20
    15
    10
    5
    5
    END PL/0
```  
* 效果图如下： 
    
    ![preview](https://github.com/penglsh/pl0/raw/master/images/run2.PNG)