###解mybatis的mapper.xml编写
http://www.jb51.net/article/82028.htm 
1.resultMap 
SQL 映射XML 文件是所有sql语句放置的地方。需要定义一个workspace，一般定义为对应的接口类的路径。写好SQL语句映射文件后，需要在MyBAtis配置文件mappers标签中引用，例如： 
"""
<mappers> 
  <mapper resource="com/liming/manager/data/mappers/UserMapper.xml" /> 
  <mapper resource="com/liming/manager/data/mappers/StudentMapper.xml" /> 
  <mapper resource="com/liming/manager/data/mappers/ClassMapper.xml" /> 
  <mapper resource="com/liming/manager/data/mappers/TeacherMapper.xml" /> 
</mappers> 
"""
