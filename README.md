# 详解mybatis 的 mapper.xml 编写
`http://www.jb51.net/article/82028.htm`
## 1.resultMap  
SQL 映射XML 文件是所有sql语句放置的地方。需要定义一个workspace，一般定义为对应的接口类的路径。写好SQL语句映射文件后，需要在MyBAtis配置文件mappers标签中引用，例如：<br/> 
```xml 
<mappers>  
<mapper resource="com/liming/manager/data/mappers/UserMapper.xml" />  
<mapper resource="com/liming/manager/data/mappers/StudentMapper.xml" />   
<mapper resource="com/liming/manager/data/mappers/ClassMapper.xml" />   
<mapper resource="com/liming/manager/data/mappers/TeacherMapper.xml" />  
</mappers>
```
当Java接口与XML文件在一个相对路径下时，可以不在myBatis配置文件的mappers中声明。 
SQL 映射XML 文件一些初级的元素：  
（1）. cache – 配置给定模式的缓存  
（2）. cache-ref – 从别的模式中引用一个缓存   
（3）. resultMap – 这是最复杂而却强大的一个元素了，它描述如何从结果集中加载对象  
（4）. sql – 一个可以被其他语句复用的SQL 块  
（5）. insert – 映射INSERT 语句   
（6）. update – 映射UPDATE 语句   
（7）. delete – 映射DELEETE 语句   
（8）. select  -  映射SELECT语句  
### 1.1 resultMap 
resultMap 是MyBatis 中最重要最强大的元素了。你可以让你比使用JDBC 调用结果集省掉90%的代码，也可以让你做许多JDBC 不支持的事。现实上，要写一个等同类似于交互的映射这样的复杂语句，可能要上千行的代码。ResultMaps的目的，就是这样简单的语句而不需要多余的结果映射，更多复杂的语句，除了只要一些绝对必须的语句描述关系以外，再也不需要其它的。 
resultMap属性：type为java实体类；id为此resultMap的标识。   
resultMap可以设置的映射：<br/>
（1）. constructor – 用来将结果反射给一个实例化好的类的构造器<br/>
　　a) idArg – ID 参数；将结果集标记为ID，以方便全局调用<br/>
　　b) arg –反射到构造器的通常结果<br/>
（2）. id – ID 结果，将结果集标记为ID，以方便全局调用<br/>
（3）. result – 反射到JavaBean 属性的普通结果<br/>
（4）. association – 复杂类型的结合；多个结果合成的类型<br/>
　　a) nested result mappings – 几resultMap 自身嵌套关联，也可以引用到一个其它上<br/>
（5）. collection –复杂类型集合a collection of complex types<br/>
（6）. nested result mappings – resultMap 的集合，也可以引用到一个其它上<br/>
（7）. discriminator – 使用一个结果值以决定使用哪个resultMap<br/>
　　a) case – 基本一些值的结果映射的case 情形<br/>
　　`i. nested result mappings –一个case 情形本身就是一个结果映射，因此也可以包括一些相同的元素，也可以引用一个外部resultMap。`<br/>
#### 1.1.1 id、result  
id、result是最简单的映射，id为主键映射；result其他基本数据库表字段到实体类属性的映射。<br/>
最简单的例子：<br/>
``` xml
<resultMap type="liming.student.manager.data.model.StudentEntity" id="studentResultMap"> 
  <id property="studentId"    column="STUDENT_ID" javaType="String" jdbcType="VARCHAR"/>  
  <result property="studentName"    column="STUDENT_NAME" javaType="String" jdbcType="VARCHAR"/>  
  <result property="studentSex"    column="STUDENT_SEX" javaType="int" jdbcType="INTEGER"/>  
  <result property="studentBirthday"  column="STUDENT_BIRTHDAY" javaType="Date" jdbcType="DATE"/>  
  <result property="studentPhoto" column="STUDENT_PHOTO" javaType="byte[]" jdbcType="BLOB" typeHandler="org.apache.ibatis.type.BlobTypeHandler" />
</resultMap>
```
id result语句属性配置细节： 
<table>
  <tr>
    <td>属性</td>
    <td>描述</td>
  </tr>
 <tr>
    <td>property</td>
    <td>需要映射到JavaBean 的属性名称。</td>
  </tr>
 <tr>
    <td>column</td>
    <td>数据表的列名或者标签别名。 </td>
  </tr>
 <tr>
    <td>javaType</td>
    <td>一个完整的类名，或者是一个类型别名。<br/>如果你匹配的是一个JavaBean，那MyBatis 通常会自行检测到。<br>然后，如果你是要映射到一个HashMap，那你需要指定javaType 要达到的目的。 </td>
  </tr>
 <tr>
    <td>jdbcType</td>
    <td>数据表支持的类型列表。<br/>这个属性只在insert,update 或delete 的时候针对允许空的列有用。<br/>JDBC 需要这项，但MyBatis 不需要。如果你是直接针对JDBC 编码，且有允许空的列，而你要指定这项。</td>
  </tr>
 <tr>
    <td>typeHandler</td>
    <td>使用这个属性可以覆写类型处理器。<br/>这项值可以是一个完整的类名，也可以是一个类型别名。 </td>
  </tr>
</table>
支持的JDBC类型 
       为了将来的引用，MyBatis 支持下列JDBC 类型，通过JdbcType 枚举： 
BIT，FLOAT，CHAR，TIMESTAMP，OTHER，UNDEFINED，TINYINT，REAL，VARCHAR，BINARY，BLOB，NVARCHAR，SMALLINT，DOUBLE，LONGVARCHAR，VARBINARY，CLOB，NCHAR，INTEGER，NUMERIC，DATE，LONGVARBINARY，BOOLEAN，NCLOB，BIGINT，DECIMAL，TIME，NULL，CURSOR 

#### 1.1.2 constructor 
我们使用id、result时候，需要定义java实体类的属性映射到数据库表的字段上。这个时候是使用JavaBean实现的。当然我们也可以使用实体类的构造方法来实现值的映射，这个时候是通过构造方法参数的书写的顺序来进行赋值的。使用construcotr功能有限（例如使用collection级联查询）。上面使用id、result实现的功能就可以改为：  
``` xml
<resultMap type="StudentEntity" id="studentResultMap" >  
 <constructor> 
    <idArg javaType="String" column="STUDENT_ID"/>    
    <arg javaType="String" column="STUDENT_NAME"/>
    <arg javaType="String" column="STUDENT_SEX"/>
    <arg javaType="Date" column="STUDENT_BIRTHDAY"/>
  </constructor>
</resultMap>
```
当然，我们需要定义StudentEntity实体类的构造方法： 
``` java
public StudentEntity(String studentID, String studentName, String studentSex, Date studentBirthday){
 
  this.studentID = studentID; 
  
  this.studentName = studentName; 
  
  this.studentSex = studentSex; 
  
  this.studentBirthday = studentBirthday; 
}
```
#### 1.1.3 association联合 
联合元素用来处理“一对一”的关系。需要指定映射的Java实体类的属性，属性的javaType（通常MyBatis 自己会识别）。对应的数据库表的列名称。如果想覆写的话返回结果的值，需要指定typeHandler。 
不同情况需要告诉MyBatis 如何加载一个联合。MyBatis 可以用两种方式加载：  
（1）. select: 执行一个其它映射的SQL 语句返回一个Java实体类型。较灵活；  
（2）. resultsMap: 使用一个嵌套的结果映射来处理通过join查询结果集，映射成Java实体类型。 
例如，一个班级对应一个班主任。 
  首先定义好班级中的班主任属性： 
``` java
private TeacherEntity teacherEntity; 
```
##### 1.1.3.1使用select实现联合 
例：班级实体类中有班主任的属性，通过联合在得到一个班级实体时，同时映射出班主任实体。 
这样可以直接复用在TeacherMapper.xml文件中定义好的查询teacher根据其ID的select语句。而且不需要修改写好的SQL语句，只需要直接修改resultMap即可。 
ClassMapper.xml文件部分内容： 
``` xml
<resultMap type="ClassEntity" id="classResultMap"> 
  <id property="classID" column="CLASS_ID" /> 
  <result property="className" column="CLASS_NAME" /> 
  <result property="classYear" column="CLASS_YEAR" /> 
  <association property="teacherEntity" column="TEACHER_ID" select="getTeacher"/> 
</resultMap> 
  
<select id="getClassByID" parameterType="String" resultMap="classResultMap"> 
  SELECT * FROM CLASS_TBL CT 
  WHERE CT.CLASS_ID = #{classID}; 
</select>
```
TeacherMapper.xml文件部分内容： 
``` xml
<resultMap type="TeacherEntity" id="teacherResultMap"> 
  <id property="teacherID" column="TEACHER_ID" /> 
  <result property="teacherName" column="TEACHER_NAME" /> 
  <result property="teacherSex" column="TEACHER_SEX" /> 
  <result property="teacherBirthday" column="TEACHER_BIRTHDAY"/> 
  <result property="workDate" column="WORK_DATE"/> 
  <result property="professional" column="PROFESSIONAL"/> 
</resultMap> 
  
<select id="getTeacher" parameterType="String" resultMap="teacherResultMap"> 
  SELECT * 
   FROM TEACHER_TBL TT 
   WHERE TT.TEACHER_ID = #{teacherID} 
</select>
```
##### 1.1.3.2使用resultMap实现联合 
与上面同样的功能，查询班级，同时查询器班主任。需在association中添加resultMap（在teacher的xml文件中定义好的），新写sql（查询班级表left join教师表），不需要teacher的select。 
修改ClassMapper.xml文件部分内容：  
``` xml
<resultMap type="ClassEntity" id="classResultMap"> 
  <id property="classID" column="CLASS_ID" /> 
  <result property="className" column="CLASS_NAME" /> 
  <result property="classYear" column="CLASS_YEAR" /> 
  <association property="teacherEntity" column="TEACHER_ID" resultMap="teacherResultMap"/> 
</resultMap> 
  
<select id="getClassAndTeacher" parameterType="String" resultMap="classResultMap"> 
  SELECT * 
   FROM CLASS_TBL CT LEFT JOIN TEACHER_TBL TT ON CT.TEACHER_ID = TT.TEACHER_ID 
   WHERE CT.CLASS_ID = #{classID}; 
</select> 
```
其中的teacherResultMap请见上面TeacherMapper.xml文件部分内容中。 
#### 1.1.4 collection聚集 
聚集元素用来处理“一对多”的关系。需要指定映射的Java实体类的属性，属性的javaType（一般为ArrayList）；列表中对象的类型ofType（Java实体类）；对应的数据库表的列名称；<br/>
不同情况需要告诉MyBatis 如何加载一个聚集。MyBatis 可以用两种方式加载：<br/>
（1）. select: 执行一个其它映射的SQL 语句返回一个Java实体类型。较灵活；<br/> 
（2）. resultsMap: 使用一个嵌套的结果映射来处理通过join查询结果集，映射成Java实体类型。<br/>
例如，一个班级有多个学生。<br/>
首先定义班级中的学生列表属性：<br/>
``` java
private List<StudentEntity> studentList;  
```
##### 1.1.4.1使用select实现聚集 
用法和联合很类似，区别在于，这是一对多，所以一般映射过来的都是列表。所以这里需要定义javaType为ArrayList，还需要定义列表中对象的类型ofType，以及必须设置的select的语句名称（需要注意的是，这里的查询student的select语句条件必须是外键classID）。 
ClassMapper.xml文件部分内容： 
``` xml
<resultMap type="ClassEntity" id="classResultMap"> 
  <id property="classID" column="CLASS_ID" /> 
  <result property="className" column="CLASS_NAME" /> 
  <result property="classYear" column="CLASS_YEAR" /> 
  <association property="teacherEntity" column="TEACHER_ID" select="getTeacher"/> 
  <collection property="studentList" column="CLASS_ID" javaType="ArrayList" ofType="StudentEntity" select="getStudentByClassID"/> 
</resultMap> 
  
<select id="getClassByID" parameterType="String" resultMap="classResultMap"> 
  SELECT * FROM CLASS_TBL CT 
  WHERE CT.CLASS_ID = #{classID}; 
</select> 
```
StudentMapper.xml文件部分内容： 
``` xml
<!-- java属性，数据库表字段之间的映射定义 --> 
<resultMap type="StudentEntity" id="studentResultMap"> 
  <id property="studentID" column="STUDENT_ID" /> 
  <result property="studentName" column="STUDENT_NAME" /> 
  <result property="studentSex" column="STUDENT_SEX" /> 
  <result property="studentBirthday" column="STUDENT_BIRTHDAY" /> 
</resultMap> 
  
<!-- 查询学生list，根据班级id --> 
<select id="getStudentByClassID" parameterType="String" resultMap="studentResultMap"> 
  <include refid="selectStudentAll" /> 
  WHERE ST.CLASS_ID = #{classID} 
</select> 
```
##### 1.1.4.2使用resultMap实现聚集 
使用resultMap，就需要重写一个sql，left join学生表。 
``` xml
<resultMap type="ClassEntity" id="classResultMap"> 
  <id property="classID" column="CLASS_ID" /> 
  <result property="className" column="CLASS_NAME" /> 
  <result property="classYear" column="CLASS_YEAR" /> 
  <association property="teacherEntity" column="TEACHER_ID" resultMap="teacherResultMap"/> 
  <collection property="studentList" column="CLASS_ID" javaType="ArrayList" ofType="StudentEntity" resultMap="studentResultMap"/> 
</resultMap> 
  
<select id="getClassAndTeacherStudent" parameterType="String" resultMap="classResultMap"> 
  SELECT * 
   FROM CLASS_TBL CT 
      LEFT JOIN STUDENT_TBL ST 
       ON CT.CLASS_ID = ST.CLASS_ID 
      LEFT JOIN TEACHER_TBL TT 
       ON CT.TEACHER_ID = TT.TEACHER_ID 
   WHERE CT.CLASS_ID = #{classID}; 
</select> 
```
其中的teacherResultMap请见上面TeacherMapper.xml文件部分内容中。studentResultMap请见上面StudentMapper.xml文件部分内容中。 
#### 1.1.5 discriminator鉴别器 
有时一个单独的数据库查询也许返回很多不同（但是希望有些关联）数据类型的结果集。鉴别器元素就是被设计来处理这个情况的，还有包括类的继承层次结构。鉴别器非常容易理解，因为它的表现很像Java语言中的switch语句。<br/>
　　定义鉴别器指定了column和javaType属性。<br/>
　　列是MyBatis查找比较值的地方。<br/>
　　JavaType是需要被用来保证等价测试的合适类型（尽管字符串在很多情形下都会有用）。<br/>
下面这个例子为，当classId为20000001时，才映射classId属性。<br/>
``` xml
<resultMap type="liming.student.manager.data.model.StudentEntity" id="resultMap_studentEntity_discriminator"> 
  <id property="studentId"    column="STUDENT_ID" javaType="String" jdbcType="VARCHAR"/> 
  <result property="studentName"    column="STUDENT_NAME" javaType="String" jdbcType="VARCHAR"/> 
  <result property="studentSex"    column="STUDENT_SEX" javaType="int" jdbcType="INTEGER"/> 
  <result property="studentBirthday"  column="STUDENT_BIRTHDAY" javaType="Date" jdbcType="DATE"/> 
  <result property="studentPhoto" column="STUDENT_PHOTO" javaType="byte[]" jdbcType="BLOB" typeHandler="org.apache.ibatis.type.BlobTypeHandler" /> 
  <result property="placeId"      column="PLACE_ID" javaType="String" jdbcType="VARCHAR"/> 
  <discriminator column="CLASS_ID" javaType="String" jdbcType="VARCHAR"> 
    <case value="20000001" resultType="liming.student.manager.data.model.StudentEntity" > 
      <result property="classId" column="CLASS_ID" javaType="String" jdbcType="VARCHAR"/> 
    </case> 
  </discriminator> 
</resultMap> 
```
## 2.增删改查、参数、缓存 
### 2.1 select 
一个select 元素非常简单。例如： 
``` xml
<!-- 查询学生，根据id --> 
<select id="getStudent" parameterType="String" resultMap="studentResultMap"> 
  SELECT ST.STUDENT_ID, 
        ST.STUDENT_NAME, 
        ST.STUDENT_SEX, 
        ST.STUDENT_BIRTHDAY, 
        ST.CLASS_ID 
     FROM STUDENT_TBL ST 
     WHERE ST.STUDENT_ID = #{studentID} 
</select> 
```
这条语句就叫做 ‘getStudent’，有一个String参数，并返回一个StudentEntity类型的对象。 
注意参数的标识是：#{studentID}。 
select 语句属性配置细节：  
<table>
  <tr>
   <td>属性</td>
   <td>描述</td>
   <td>取值</td>
   <td>默认</td>
  </tr>
  <tr>
   <td>id</td>
   <td>在这个模式下唯一的标识符，可被其它语句引用</td>
   <td>  </td>
   <td>  </td>
  </tr>
  <tr>
   <td>parameterType</td>
   <td>传给此语句的参数的完整类名或别名</td>
   <td>  </td>
   <td>  </td>
  </tr>
  <tr>
   <td>resultType</td>
   <td>语句返回值类型的整类名或别名。注意，如果是集合，那么这里填写的是集合的项的整类名或别名，而不是集合本身的类名。（resultType 与resultMap 不能并用）</td>
   <td>  </td>
   <td>  </td>
  </tr>
  <tr>
   <td>resultMap</td>
   <td>引用的外部resultMap 名。结果集映射是MyBatis 中最强大的特性。许多复杂的映射都可以轻松解决。（resultType 与resultMap 不能并用）</td>
   <td>  </td>
   <td>  </td>
  </tr>
  <tr>
   <td>flushCache</td>
   <td>如果设为true，则会在每次语句调用的时候就会清空缓存。select 语句默认设为false</td>
   <td>true<br>false</td>
   <td>false</td>
  </tr>   
  <tr>
   <td>useCache</td>
   <td>如果设为true，则语句的结果集将被缓存。select 语句默认设为false</td>
   <td>true<br>false</td>
   <td>false</td>
  </tr>
  <tr>
   <td>timeout</td>
   <td>设置驱动器在抛出异常前等待回应的最长时间，默认为不设值，由驱动器自己决定</td>
   <td>正整数</td>
   <td>未设置</td>
  </tr>
  <tr>
   <td>fetchSize</td>
   <td>设置一个值后，驱动器会在结果集数目达到此数值后，激发返回，默认为不设值，由驱动器自己决定</td>
   <td>正整数</td>
   <td>驱动器决定</td>
  </tr>
  <tr>
   <td>statementType</td>
   <td>statement，preparedstatement，callablestatement。<br/>普通sql语句、预准备语句、可调用语句 </td>
   <td>STATEMENT<br>PREPARED<br>CALLABLE</td>
   <td>PREPARED</td>
  </tr>
  <tr>
   <td>resultSetType</td>
   <td>forward_only，scroll_sensitive，scroll_insensitive。<br/>只转发，滚动敏感，不区分大小写的滚动</td>
   <td>FORWARD_ONLY<br>SCROLL_SENSITIVE<br>SCROLL_INSENSITIVE</td>
   <td>驱动器决定</td>
  </tr>
</table>
  
### 2.2 insert  
一个简单的insert语句  
``` xml  
<!-- 插入学生 --> 
<insert id="insertStudent" parameterType="StudentEntity"> 
    INSERT INTO STUDENT_TBL (STUDENT_ID, 
                     STUDENT_NAME, 
                     STUDENT_SEX, 
                     STUDENT_BIRTHDAY, 
                     CLASS_ID) 
       VALUES  (#{studentID}, 
             #{studentName}, 
             #{studentSex}, 
             #{studentBirthday}, 
             #{classEntity.classID}) 
</insert> 
```
insert可以使用数据库支持的自动生成主键策略，设置useGeneratedKeys=”true”，然后把keyProperty 设成对应的列，就搞定了。比如说上面的StudentEntity 使用auto-generated 为id 列生成主键.还可以使用selectKey元素。下面例子，使用mysql数据库nextval('student')为自定义函数，用来生成一个key。  
``` xml  
<!-- 插入学生 自动主键--> 
<insert id="insertStudentAutoKey" parameterType="StudentEntity"> 
  <selectKey keyProperty="studentID" resultType="String" order="BEFORE"> 
      select nextval('student') 
  </selectKey> 
    INSERT INTO STUDENT_TBL (STUDENT_ID, 
                 STUDENT_NAME, 
                 STUDENT_SEX, 
                 STUDENT_BIRTHDAY, 
                 CLASS_ID) 
       VALUES  (#{studentID}, 
            #{studentName}, 
            #{studentSex}, 
            #{studentBirthday}, 
            #{classEntity.classID})   
</insert> 
```
insert语句属性配置细节：  
<table>
  <tr>
    <td>属性</td>
    <td>描述</td>
    <td>取值</td>
    <td>默认</td>
  </tr>
  <tr>
    <td>id</td>
    <td>在这个模式下唯一的标识符，可被其它语句引用</td>
    <td>  </td>
    <td>  </td>
  </tr>
  <tr>
    <td>parameterType</td>
    <td>传给此语句的参数的完整类名或别名</td>
    <td>  </td>
    <td>  </td>
  </tr>
  <tr>
    <td>flushCache</td>
    <td>如果设为true，则会在每次语句调用的时候就会清空缓存。<br/>select 语句默认设为false</td>
    <td>true<br>false</td>
    <td>false</td>
  </tr>
  <tr>
    <td>useCache</td>
    <td>如果设为true，则语句的结果集将被缓存。<br/>select 语句默认设为false</td>
    <td>true<br>false</td>
    <td>false</td>
  </tr>
  <tr>
    <td>timeout</td>
    <td>设置驱动器在抛出异常前等待回应的最长时间，默认为不设值，由驱动器自己决定</td>
    <td>正整数</td>
    <td>未设置</td>
  </tr>
  <tr>
    <td>fetchSize</td>
    <td>设置一个值后，驱动器会在结果集数目达到此数值后，<br/>激发返回，默认为不设值，由驱动器自己决定</td>
    <td>正整数</td>
    <td>驱动器决定</td>
  </tr>
  <tr>
   <td>statementType</td>
   <td>statement，preparedstatement，callablestatement。<br/>普通sql语句、预准备语句、可调用语句 </td>
   <td>STATEMENT<br>PREPARED<br>CALLABLE</td>
   <td>PREPARED</td>
  </tr>
  <tr>
    <td>useGeneratedKeys</td>
    <td>告诉MyBatis 使用JDBC 的getGeneratedKeys 方法来获取数据库自己生成的主键（MySQL、SQLSERVER 等 
关系型数据库会有自动生成的字段）。<br/>默认：false </td>
    <td>true<br>false</td>
    <td>false</td>
  </tr>
  <tr>
   <td>keyProperty</td>
   <td>标识一个将要被MyBatis 设置进getGeneratedKeys 的key 所返回的值，<br/>或者为insert 语句使用一个selectKey 
子元素。 </td>
   <td>  </td>
   <td>  </td>
  </tr>
</table> 
  
selectKey语句属性配置细节：  
<table>
  <tr>
    <td>属性</td>
    <td>描述</td>
    <td>取值</td>    
  </tr>
  <tr>
    <td>keyProperty</td>
    <td>selectKey 语句生成结果需要设置的属性。</td>
    <td>  </td>    
  </tr>
  <tr>
    <td>resultType</td>
    <td>生成结果类型，MyBatis 允许使用基本的数据类型，包括String 、int类型。	</td>
    <td>  </td>    
  </tr>
  <tr>
    <td>order</td>
    <td>可以设成BEFORE 或者AFTER，如果设为BEFORE，那它会先选择主键，然后设置keyProperty，再执行insert语句；<br/>如果设为AFTER，它就先运行insert 语句再运行selectKey 语句，通常是insert 语句中内部调用数据库（像Oracle）内嵌的序列机制。</td>
    <td>BEFORE<br>AFTER </td>    
  </tr>
  <tr>
    <td>statementType</td>
    <td>像上面的那样， MyBatis 支持STATEMENT，PREPARED和CALLABLE 的语句形式， 对应Statement ，PreparedStatement 和CallableStatement 响应	CALLABLE</td>
    <td>STATEMENT<br>PREPARED<br>CALLABLE</td>    
  </tr>
</table> 
  
### 2.3 update、delete 
一个简单的update：
``` xml
<!-- 更新学生信息 --> 
<update id="updateStudent" parameterType="StudentEntity"> 
    UPDATE STUDENT_TBL 
      SET STUDENT_TBL.STUDENT_NAME = #{studentName},  
        STUDENT_TBL.STUDENT_SEX = #{studentSex}, 
        STUDENT_TBL.STUDENT_BIRTHDAY = #{studentBirthday}, 
        STUDENT_TBL.CLASS_ID = #{classEntity.classID} 
     WHERE STUDENT_TBL.STUDENT_ID = #{studentID};   
</update> 
```
一个简单的delete：  
``` xml
<!-- 删除学生 --> 
<delete id="deleteStudent" parameterType="StudentEntity"> 
    DELETE FROM STUDENT_TBL WHERE STUDENT_ID = #{studentID} 
</delete> 
```
update、delete语句属性配置细节：  
<table>
  <tr>
    <td>属性</td>
    <td>描述</td>
    <td>取值</td>
    <td>默认</td>
  </tr>
  <tr>
    <td>id</td>
    <td>在这个模式下唯一的标识符，可被其它语句引用</td>
    <td>  </td>
    <td>  </td> 
  </tr>
  <tr>
    <td>parameterType</td>
    <td>传给此语句的参数的完整类名或别名</td>
    <td>  </td>
    <td>  </td> 
  </tr> 
  <tr>
    <td>flushCache</td>
    <td>如果设为true，则会在每次语句调用的时候就会清空缓存。<br/>select 语句默认设为false</td>
    <td>true<br>false</td>
    <td>false</td> 
  </tr> 
  <tr>
    <td>useCache</td>
    <td>如果设为true，则语句的结果集将被缓存。<br>select 语句默认设为false</td>
    <td>true<br>false</td>
    <td>false</td> 
  </tr>
  <tr>
    <td>timeout</td>
    <td>设置驱动器在抛出异常前等待回应的最长时间，默认为不设值，由驱动器自己决定</td>
    <td>正整数</td>
    <td>未设置</td> 
  </tr> 
  <tr>
    <td>fetchSize</td>
    <td>设置一个值后，驱动器会在结果集数目达到此数值后，激发返回，默认为不设值，由驱动器自己决定</td>
    <td>正整数</td>
    <td>驱动器决定</td> 
  </tr> 
  <tr>
    <td>statementType</td>
    <td>像上面的那样， MyBatis 支持STATEMENT，PREPARED和CALLABLE 的语句形式， 对应Statement ，PreparedStatement 和CallableStatement 响应	CALLABLE</td>
    <td>STATEMENT<br>PREPARED<br>CALLABLE</td>    
  </tr>
</table>
  
### 2.4 sql 
Sql元素用来定义一个可以复用的SQL 语句段，供其它语句调用。比如： 
``` xml
<!-- 复用sql语句 查询student表所有字段 --> 
<sql id="selectStudentAll"> 
    SELECT ST.STUDENT_ID, 
          ST.STUDENT_NAME, 
          ST.STUDENT_SEX, 
          ST.STUDENT_BIRTHDAY, 
          ST.CLASS_ID 
       FROM STUDENT_TBL ST 
</sql> 
```
这样，在select的语句中就可以直接引用使用了，将上面select语句改成： 
``` xml
<!-- 查询学生，根据id --> 
<select id="getStudent" parameterType="String" resultMap="studentResultMap"> 
  <include refid="selectStudentAll"/> 
      WHERE ST.STUDENT_ID = #{studentID}  
</select>
```

### 2.5 parameters 
上面很多地方已经用到了参数，比如查询、修改、删除的条件，插入，修改的数据等，MyBatis可以使用的基本数据类型和Java的复杂数据类型。 
基本数据类型，String，int，date等。 但是使用基本数据类型，只能提供一个参数，所以需要使用Java实体类，或Map类型做参数类型。通过#{}可以直接得到其属性。 

#### 2.5.1 基本类型参数 
根据入学时间，检索学生列表： 
``` xml
<!-- 查询学生list，根据入学时间 --> 
<select id="getStudentListByDate" parameterType="Date" resultMap="studentResultMap"> 
  SELECT * 
   FROM STUDENT_TBL ST LEFT JOIN CLASS_TBL CT ON ST.CLASS_ID = CT.CLASS_ID 
   WHERE CT.CLASS_YEAR = #{classYear};   
</select> 
```
``` java 
List<StudentEntity> studentList = studentMapper.getStudentListByClassYear(StringUtil.parse("2007-9-1")); 
for (StudentEntity entityTemp : studentList) { 
  System.out.println(entityTemp.toString()); 
} 
```
#### 2.5.2 Java实体类型参数 
根据姓名和性别，检索学生列表。使用实体类做参数： 
``` xml
<!-- 查询学生list，like姓名、=性别，参数entity类型 --> 
<select id="getStudentListWhereEntity" parameterType="StudentEntity" resultMap="studentResultMap"> 
  SELECT * from STUDENT_TBL ST 
    WHERE ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%') 
     AND ST.STUDENT_SEX = #{studentSex} 
</select> 
```
``` java
StudentEntity entity = new StudentEntity(); 
entity.setStudentName("李"); 
entity.setStudentSex("男"); 
List<StudentEntity> studentList = studentMapper.getStudentListWhereEntity(entity); 
for (StudentEntity entityTemp : studentList) { 
  System.out.println(entityTemp.toString()); 
} 
```
#### 2.5.3 Map参数 
根据姓名和性别，检索学生列表。使用Map做参数： 
``` xml
<!-- 查询学生list，=性别，参数map类型 --> 
<select id="getStudentListWhereMap" parameterType="Map" resultMap="studentResultMap"> 
  SELECT * from STUDENT_TBL ST 
   WHERE ST.STUDENT_SEX = #{sex} 
     AND ST.STUDENT_SEX = #{sex} 
</select> 
```
``` java
Map<String, String> map = new HashMap<String, String>(); 
map.put("sex", "女"); 
map.put("name", "李"); 
List<StudentEntity> studentList = studentMapper.getStudentListWhereMap(map); 
for (StudentEntity entityTemp : studentList) { 
  System.out.println(entityTemp.toString()); 
} 
```
#### 2.5.4 多参数的实现
如果想传入多个参数，则需要在接口的参数上添加@Param注解。给出一个实例：  
接口写法：  
``` java
public List<StudentEntity> getStudentListWhereParam(@Param(value = "name") String name, @Param(value = "sex") String sex, @Param(value = "birthday") Date birthdar, @Param(value = "classEntity") ClassEntity classEntity); 
```
SQL写法： 
``` xml
<!-- 查询学生list，like姓名、=性别、=生日、=班级，多参数方式 --> 
<select id="getStudentListWhereParam" resultMap="studentResultMap"> 
  SELECT * from STUDENT_TBL ST 
  <where> 
    <if test="name!=null and name!='' "> 
      ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{name}),'%') 
    </if> 
    <if test="sex!= null and sex!= '' "> 
      AND ST.STUDENT_SEX = #{sex} 
    </if> 
    <if test="birthday!=null"> 
      AND ST.STUDENT_BIRTHDAY = #{birthday} 
    </if> 
    <if test="classEntity!=null and classEntity.classID !=null and classEntity.classID!='' "> 
      AND ST.CLASS_ID = #{classEntity.classID} 
    </if> 
  </where> 
</select> 
```
进行查询： 
``` java
List<StudentEntity> studentList = studentMapper.getStudentListWhereParam("", "",StringUtil.parse("1985-05-28"), classMapper.getClassByID("20000002")); 
for (StudentEntity entityTemp : studentList) { 
  System.out.println(entityTemp.toString()); 
} 
```
#### 2.5.5字符串代入法 
　　默认的情况下，使用#{}语法会促使MyBatis 生成PreparedStatement 属性并且使用PreparedStatement 的参数（=？）来安全的设置值。<br>尽管这些是快捷安全，也是经常使用的，但有时候你可能想直接未更改的字符串代入到SQL 语句中。<br>比如说，对于ORDER BY，你可能会这样使用：ORDER BY ${columnName}但MyBatis 不会修改和规避掉这个字符串。   
　　注意：这样地接收和应用一个用户输入到未更改的语句中，是非常不安全的。这会让用户能植入破坏代码，所以，要么要求字段不要允许客户输入，要么你直接来检测他的合法性 。   
### 2.6 cache缓存 
　　MyBatis 包含一个强在的、可配置、可定制的缓存机制。MyBatis 3 的缓存实现有了许多改进，既强劲也更容易配置。<br>默认的情况，缓存是没有开启，除了会话缓存以外，它可以提高性能，且能解决全局依赖。开启二级缓存，你只需要在SQL 映射文件中加入简单的一行：`<cache/>`<br> 
这句简单的语句的作用如下：<br>
（1）. 所有在映射文件里的select 语句都将被缓存。<br>
（2）. 所有在映射文件里insert,update 和delete 语句会清空缓存。<br> 
（3）. 缓存使用“最近很少使用”算法来回收<br>
（4）. 缓存不会被设定的时间所清空。<br>
（5）. 每个缓存可以存储1024 个列表或对象的引用（不管查询出来的结果是什么）。<br>
（6）. 缓存将作为“读/写”缓存，意味着获取的对象不是共享的且对调用者是安全的。不会有其它的调用<br>
（7）. 者或线程潜在修改。<br>
例如，创建一个FIFO 缓存让60 秒就清空一次，存储512 个对象结果或列表引用，并且返回的结果是只读。<br>因为在不用的线程里的两个调用者修改它们可能会导致引用冲突。 
``` xml
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"> 
</cache>
```
还可以在不同的命名空间里共享同一个缓存配置或者实例。在这种情况下，你就可以使用cache-ref 来引用另外一个缓存。 
`<cache-ref namespace="com.liming.manager.data.StudentMapper"/>`
Cache 语句属性配置细节： 
<table>
  <tr>
    <td>属性</td>
    <td>描述</td>
    <td>取值</td>
    <td>默认</td>
  </tr>
  <tr>
    <td>eviction</td>
    <td>
　　缓存策略：<br>  
LRU - 最近最少使用法：移出最近较长周期内都没有被使用的对象。<br>
FIFI- 先进先出：移出队列里较早的对象<br>
SOFT - 软引用：基于软引用规则，使用垃圾回收机制来移出对象<br>
WEAK - 弱引用：基于弱引用规则，使用垃圾回收机制来强制性地移出对象
    </td>
    <td>  
LRU<br>
FIFI<br>
SOFT<br>
WEAK<br>　
　　</td>
    <td>
LRU
　　</td>
  </tr>
  <tr>
    <td>flushInterval</td>
    <td>代表一个合理的毫秒总计时间。默认是不设置，因此使用无间隔清空即只能调用语句来清空。</td>
    <td>正整数</td>
    <td>不设置</td>
  </tr>
  <tr>
    <td>size</td>
    <td>缓存的对象的大小</td>
    <td>正整数</td>
    <td>1024</td>
  </tr>  
  <tr>
    <td>readOnly</td>
    <td>
只读缓存将对所有调用者返回同一个实例。<br>因此都不能被修改，这可以极大的提高性能。<br>
可写的缓存将通过序列化来返回一个缓存对象的拷贝。<br>这会比较慢，但是比较安全。所以默认值是false。 
　　</td>
    <td>true<br>false</td>
    <td>false </td>
  </tr>
</table>
  

	
