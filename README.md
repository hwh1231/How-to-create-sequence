# How-to-create-sequence

create sequence sequenceName
创建序列的命令为CREATE SEQUENCE,它的完整语法格式为：

CREATE SEQUENCE 序列名

INCREMENT BY n

START WITH n

MINVALUE n | NOMINVALUE

MAXVALUE n | NOMAXVALUE

CYCLE | NOCYCLE

CACHE | NOCACHE

ORDER | NOORDER

在这个命令的语法格式中，除序列名以外，其余各选项都是可选的。

各选项中的n是一个整数。

其中START WITH选项指定序列中的序号从哪个数字开始，默认情况下从它的最小值开始。
INCREMENT BY选项指定了序列中序号递增的幅度，也就是后一个序号比前一个序号大多少。
序号可以递增，也可以递减，所以INCREMENT BY选项中的数字n可以是正整数，也可以是负整数。
MAXVALUE用来指定序列中序号的最大值。

如果没有最大值，可用NOMAXVALUE选项代替这个选项。
同样， MINVALUE用来指定序列中序号的最小值，序列中的最小值必须小于或等于它的开始值。
如果为序列指定了最大值，那么当序列中的序号被悄耗完时，用户将无法从这个序列中取得序号。
选项CYCLE使得序列中的序号可以循环使用。

当用户正在使用序列中的最大值时，下一个可以使用的序号就是它的最小值。
用户每使用序列一次，都要对序列进行一次查询。

如果把序列中的序号放在内存中进行缓冲，那么用户获得序号的速度将大大加快。
选项CACHE的作用就是将序列中接下来的n个序号在内存中进行缓冲。
如果不希望进行缓冲，可以用NOCACHE选项代替它。

序列在创建之后，在使用的过程中，可以对其进行修改。

比如修改它的最大值、最小值、增幅等，但是不能修改开始值。
需要注意的是，如果已经有部分序号被使用，那么对序列的修改只影响以后的序号，对以前已经使用的序号不起作用。

修改序列的命令是ALTER SEQUENCE。

用户可以修改自己的序列，如果希望修改其他用户的序列，则需要具有ALTER ANY SEQUENCE这个系统权限。
ALTER SEQUENCE命令的用法与CREATE SEQUENCE命令的用法基本相同，只要将关键字CREATE替换为ALTER即可。

删除序列的命令是DROP SEQUENCE。

用户可以删除自己创建的序列，如果要删除其他用户的序列，则要具有DROP ANY SEQUENCE 系统权限。
序列被删除后，它的相关信息就被从数据字典中删除。


序列的使用

对用户而言，序列中的可用资源是其中包含的序号。

用户可以通过SELECT命令获得可用的序号，也可以将序号应用于DML语句和表达式中。
如果要使用其他用户的序列，则 必须具有对该序列的SELECT权限。

序列提供了两个伪列，即NEXTVAL 和CURRVAL,用来访问序列中的序号。
其中NEXTVAL代表下一个可用的序号， CURRYAL代表当前的序号。
序列可以认为是包含了一系列序号的一个指针。
序列刚被创建时，这个指针位于第一个序号之前，以后每获得一个序号，指针就向后移动一个位置，
这时就可以用CURRVAL访问序列中的当前序号，用NEXTVAL访问下一个序号。

在第一次使用序列中的序号时，必须首先访问NEXTVAL伪列，使指针指向第一个序号。

 

通过SELECT语句可以从序列中获得一个可用的序号。

例如：

SELECT seq1.nextval FROM dual;

在SELECT语句中使用表dual是必要的，因为SELECT语句将根据表中数据的行数返回若干个序号，并且每访问一次NEXTVAL 伪列，指针就向后移动一个序号。

CURRVAL伪列代表序列中的当前序号，访问这个伪列时指针并不向后移动。

CURRVAL伪列的引用方法与NEXTVAL伪列相同，引用格式为：序列名.currval 。
序列还可应用于SELECT语句的其他形式。

例如：

SELECT seq1.nextval, empno FROM scott.emp;

在更多情况下序列的作用为表中的主键列或其他列提供一个唯一的序号。

例如：

INSERT INTO scott.dept(deptno, dname) VALUES(seq1.nextval, 'lili');

序列是一种共享式的数据库对象，用户可以直接使用自己创建的序列，其他用户也可以访问当前用户的序列，只要具有对这个序列的SELECT权限即可。
如果一个序号被某个用户获得，那么其他用户就不能再获得这个序号了。
也就是说，序列是可以共享的，但序列中的序号却是不能共享的。
对序列中序号的访问操作是作为一个单独的事务实现的，这个事务的执行与其他事务的执行成功与否无关。
如果包含一条DML语句的事务被回滚了，那么对序列的操作是无法回滚的。

注：如果是在可循环的序列中，序号可被其他用户循环使用。

在访问序列中的序号时，可能会发生序号不连续的情况，不连续的原因可能是事务发生了回滚，或者多个用户共同访问同一个序列。
一个用户要访问其他用户的序列时，不仅要具有对这个序列的SELECT权限，在访问时还要在序列的名称前以用户名进行限定。
例如：

SELECT sys.seq1.nextval FROM dual;

如果要将一个序列的SELECT权限授予其他用户，相应的GRANT命令格式为：

GRANT SELECT ON 序列名 TO 用户名;

序列信息的查询

序列作为一种数据库对象，它的相关信息也存储在数据字典中。

与序列相关的数据字典有三个： USER_SEQUENCES 、ALL SEQUENCES和DBA_SEQUENCES 。
其中数据字典USER SEQUENCES 的各列及意义如下所示：

SEQUENCE_NAME 序列名

MIN_VALUE 最小值

MAX_VALUE 最大值

INCREMENT_BY 增幅

CYCLE_FLAG 是否循环

ORDER_FLAG 是否按顺序

CACHE_SIZE 缓存大小

LAST_NUMBER 下一个可用值

注：ORDER_FLAG的解释：

Note that the ORDER option is only necessary to guarantee ordered generation if you are using Oracle with the Parallel Server option in parallel mode.
If you are using exclusive mode, sequence numbers are always generated in order.
LAST_NUMBER的解释：

使用的或缓存的最后一个序列号，一般大于缓冲区中的最后一个值。

LAST_NUMBER列在普通的数据库操作过程中不被更新，它在数据库的重新启动/恢复操作中使用。

CACHE_SIZE的解释：

在内存中缓存的序号用完之后，再向内存中添加序号。

例如：

SELECT order_flag, last_number,max_value,cache_size from user_sequences where sequence_name='SEQ1';
