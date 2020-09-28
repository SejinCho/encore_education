---
typora-copy-images-to: image
---

## 9/28(월)

#### mapreduce 기본연습

--------

> 예제1

1. 첨부 파일을 HDFS의 input폴더를 작성하고 put한 후 작업합니다.

​    출력결과 :/mywork/nasdaq

​    패키지명 : mapred.exam.stock

​    StockMapper.java, StockReducer,StockDriver.java



​	exchange => 거래구분

​	stock_symbol =>주식종목명(약칭)

​	date => 거래일자

​	stock_price_open => 시가

​	stock_price_high =>최고가

​	stock_price_low =>최저가

​	stock_price_close => 종가

​	stock_volume =>거래량

​	stock_price_adj_close =>조정금액



​	[문제] 상승마감한 것들이 년도별로 몇 건인지 조회하세요

​	[결과]

<img src="image/image-20200928102355196.png" alt="image-20200928102355196" style="zoom:80%;" />





- 하둡에서 파일 다운로드(이클립스)

![image-20200928102328333](image/image-20200928102328333.png)



- input 폴더에 파일 넣기

![image-20200928102259176](image/image-20200928102259176.png)



- 패키지, class 생성

![image-20200928105736119](image/image-20200928105736119.png)



- StockMapper.java

```java
package mapred.exam.stock;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class StockMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	private final static IntWritable one = new IntWritable(1);
	private Text outputKey = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		
		if(key.get()>0) { //제목행을 제외하고 작업을 수행하겠다는 의미
			String[] line = value.toString().split(","); //string 상태
			if(line!=null && line.length>0) {
				//연도 데이터를 output data의 키로 설정
				outputKey.set(line[2].substring(0,4));
				
				//상승마감을 판단해서 값을 추출(종가 - 시가)
				float resultValue = Float.parseFloat(line[6]) -
									Float.parseFloat(line[3]);
				
				if(resultValue > 0) { //상승마감 상태  - shuffle단으로 내보내기
					context.write(outputKey, one);	
				}
			}
		}
	}
}
```



- StockReducer.java

```java
package mapred.exam.stock;


import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

//book , [1,1,1,1...]
//a, [1,1,1,1...]
public class StockReducer extends Reducer<Text, IntWritable, Text, IntWritable>{

	//결과값을 저장할 변수
	private IntWritable resultVal = new IntWritable();
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,
			Reducer<Text, IntWritable, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		int sum = 0;
		
		//합산하는 작업
		for(IntWritable value : values) {
			sum = sum + value.get();	
		}
		resultVal.set(sum) ; //더한 결과를 resultValue에 세팅
		//reduce의 실행결과를 context에 저장
		context.write(key, resultVal);	
	}
}
```



- StockDriver.java

```java
package mapred.exam.stock;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

public class StockDriver {
	public static void main(String[] args) throws Exception {
		//1.맵리듀스를 실행하기 위한 job을 생성 
		Configuration conf = new Configuration();
		//conf 뒤에는 실행할 job의 이름 
		Job job = new Job(conf,"stock");
		
		//2. job을 처리할 실제 클래스에 대한 정보를 정의(Mapper, Reducer, Driver)
		job.setMapperClass(StockMapper.class);
		job.setReducerClass(StockReducer.class);
		job.setJarByClass(StockDriver.class); //드라이버 클래스
		
		//3. input 데이터와 output데이터의 포멧을 정의(hdfs에 텍스트 파일의 형태로 input/output이 들어감)
		job.setInputFormatClass(TextInputFormat.class);
		job.setOutputFormatClass(TextOutputFormat.class);
		
		//4. 리듀서의 출력데이터에 대한 key와 value의 타입을 명시
		job.setOutputKeyClass(Text.class);  //실제 클래스 정보를 매칭시키기 위해 .class
		job.setOutputValueClass(IntWritable.class); 
		
		//5. hdfs에 저장된 파일을 읽어오고 처리결과를 저장할 수 있도록 path정보를 설정
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		//6. 1번부터 5번까지의 설정한 내용을 기반으로 실제 job이 실행될 수 있도록 명령
		job.waitForCompletion(true);
		
	}
}
```



- jar 파일 하둡에 넣기

![image-20200928104717256](image/image-20200928104717256.png)



- jar 파일 실행

![image-20200928105520617](image/image-20200928105520617.png)



- 결과

<img src="image/image-20200928110809861.png" alt="image-20200928110809861" style="zoom:80%;" />



- cf) 파일 지우기
  - myword로 시작하는 모든 폴더 지우기

![image-20200928110631389](image/image-20200928110631389.png)





> 예제 2

- 파일 다운로드

https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/HG7NV7

![image-20200928111558262](image/image-20200928111558262.png)



- 데이터에 대한 정리

http://stat-computing.org/dataexpo/2009/the-data.html



[문제]

**HDFS경로**

**입력 : /input/1987.csv**

**출력 : /mywork/air_result_1**

**=> hdfs에 디렉토리가 없으면 디렉토리를 만들고 작업합니다.**

**에 접속하여 1987년 데이터를 월별로 항공 출발 지연 데이터가 몇 건인지 구하세요. AirMapper, AirReducer, AirDriver(mapred.exam.air)**

[결과]

![image-20200928112014090](image/image-20200928112014090.png)





- 파일 하둡에 넣기

![image-20200928112227550](image/image-20200928112227550.png)

![image-20200928112238778](image/image-20200928112238778.png)



- input 폴더로 이동

![image-20200928112754298](image/image-20200928112754298.png)



- 패키지 생성

![image-20200928113952881](image/image-20200928113952881.png)



- AirMapper.java

```
package mapred.exam.air;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class AirMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	private final static IntWritable one = new IntWritable(1);
	private Text outputKey = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		if(key.get()>0) { //제목행을 제외하고 작업을 수행
			String[] line = value.toString().split(",");
			if(line!=null && line.length > 0) {

				if(! line[15].equals("NA")) {
					//월 데이터를 output data의 키로 설정
					outputKey.set(line[1]);
					//항공 출별 지연 데이터
					int resultValut = Integer.parseInt(line[15]);
					if(resultValut>0) {
						context.write(outputKey, one);
					}
				}
				
			}
		}
	}
}
```



- AirReducer.java

```
package mapred.exam.air;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class AirReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
	
	private IntWritable resultVal = new IntWritable();
	
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,
			Reducer<Text, IntWritable, Text, IntWritable>.Context context) 
			throws IOException, InterruptedException {
		int sum = 0;
		
		//합산하는 작업
		for(IntWritable value : values) {
			sum = sum + value.get();
		}
		resultVal.set(sum); 
		context.write(key, resultVal);
	}	
}
```



- AirDriver.java

```
package mapred.exam.air;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

public class AirDriver {
	public static void main(String[] args) throws Exception {
		//1.맵리듀스를 실행하기 위한 job을 생성 
		Configuration conf = new Configuration();
		//conf 뒤에는 실행할 job의 이름 
		Job job = new Job(conf,"air");
		
		//2. job을 처리할 실제 클래스에 대한 정보를 정의(Mapper, Reducer, Driver)
		job.setMapperClass(AirMapper.class);
		job.setReducerClass(AirReducer.class);
		job.setJarByClass(AirDriver.class); //드라이버 클래스
		
		//3. input 데이터와 output데이터의 포멧을 정의(hdfs에 텍스트 파일의 형태로 input/output이 들어감)
		job.setInputFormatClass(TextInputFormat.class);
		job.setOutputFormatClass(TextOutputFormat.class);
		
		//4. 리듀서의 출력데이터에 대한 key와 value의 타입을 명시
		job.setOutputKeyClass(Text.class);  //실제 클래스 정보를 매칭시키기 위해 .class
		job.setOutputValueClass(IntWritable.class); 
		
		//5. hdfs에 저장된 파일을 읽어오고 처리결과를 저장할 수 있도록 path정보를 설정
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		//6. 1번부터 5번까지의 설정한 내용을 기반으로 실제 job이 실행될 수 있도록 명령
		job.waitForCompletion(true);
		
	}
}
```



- jar 파일 하둡에 넣기

![image-20200928121150997](image/image-20200928121150997.png)



- 실행

![image-20200928114311016](image/image-20200928114311016.png)



- 결과

![image-20200928121207831](image/image-20200928121207831.png)



------------------

#### Advanced mapreduce

-------------

- 프로젝트 생성(AdvancedMapreduceTest - java project)
  - mapreduce.exam.option.stock 패키지 생성 (안의 내용은 mapred.exam.stock의 내용 복사)
  - build.xml파일 복사

![image-20200928143255353](image/image-20200928143255353.png)



- Build Path - Configure Build Path

![image-20200928142920273](image/image-20200928142920273.png)



- add External JARs... - hadoop-core-1.2.1 jar 파일 추가

![image-20200928143018400](image/image-20200928143018400.png)



> 사용자 정의 옵션 활용

1. GevericOptionParser 활용

- hadoop을 실행할 때 - D옵션과 함께 속성 = 속성값을 입력하면 Mapper에서 이 정보를 사용할 수 있도록 정의

- commons-cli.xxx.jar 라이브러리를 추가

  1) mapper 작성

  - mapper가 실행될 때 한 번 만 실행되는 메소드
  - 하둡을 실행할 때 -D옵션과 함께 입력한 속성명을 지정하면 입력했던 속성값을 추출할 수 있다.
  - Configuration객체를 이용해서 작업
  
  2) Reducer 
  
  - 동일
  
  3) Driver
  
  - 실행할 때 사용자가 입력한 옵션을 이용할 수 있도록 설정해야하므로 기존 방식을 모두 변경 
  
  - 사용자정의 옵션을 사용하기 위한 작업(command line에 사용자가 입력하는 옵션을 mapper가 사용할 수 있도록 하기 위한 조건이 있어야 한다.)
  
    - (조건) Driver 클래스를 정의할 때 Tool인터페이스를 구현해야함 (Tool인터페이스를 상속)
  
    - (조건) Configured 클래스를 상속
  
    - (조건) Tool인터페이스의 run메소드를 오버라이딩 함
  
      => 기존의 Driver에서 정의했던 내용을 구현
  
      => GenericOptionParser를 활용해서 사용자가 입력한 명령행 매개변수에서 -D옵션과 함께 입력한 값을 분리해서 Mapper로 전달하도록 작성
  
      => 나머지 입력 값을 기존의 명령행 매개변수와 동일하게 처리하는 작업을 해야한다.
  
      
  
    - (조건) ToolRunner의 run메소드를 호출해서 Driver에 구현한 run메소드를 실행하도록 작성한다.
  
      =>하둡을 실행하면서 명령행에 입력하는 옵션을 Mapper에 전달하기 위해 ToolRunner의 run메소드를 호출해서 작업 
  
      =>ToolRunner.run(Configuration conf, Tool tool, String[] args);
  
      ​                                                                   ---------------->기존 Driver를 Tool type으로 변경
  
      =>ToolRunner 클래스의 run메소드 내부에서 Tool 객체의 run을 호출해서 실행하므로 스펙이 맞게 작성해 놓아야 한다.



**1.commons-cil.xxx.jar 라이브러리 추가**

- 하둡에서 jar파일 위치 확인 

![image-20200928151414523](image/image-20200928151414523.png)



- jar를 이클립스로 copy 

![image-20200928151507941](image/image-20200928151507941.png)



- c드라이브 sts작업파일에 붙여넣기

![image-20200928151552735](image/image-20200928151552735.png)



- 빌드패스에 추가 

![image-20200928151653324](image/image-20200928151653324.png)



**1-1) mapper 작성**

- StockOptionMapper.java에서 override
  - 주어진 옵션값에 따라 수행되는 mapper가 달라짐

![image-20200928144436484](image/image-20200928144436484.png)



- setup 메소드 선택

![image-20200928144400795](image/image-20200928144400795.png)



- StockOptionMapper.jvav

```java
package mapreduce.exam.option.stock;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class StockOptionMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	private final static IntWritable one = new IntWritable(1);
	private Text outputKey = new Text();
	
	private String jobType; //하둡실행할 때 입력한 옵션값
	
	@Override
	protected void setup(Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		
		//context.getConfiguration().get("옵션명"); -D옵션과 같이 입력하는 옵션명(옵션값과 같은 이름으로 주지 않아도 된다)
		jobType = context.getConfiguration().get("jobType");
		
	}

	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		
		if(key.get()>0) { //제목행을 제외하고 작업을 수행하겠다는 의미
			String[] line = value.toString().split(","); //string 상태
			if(line!=null && line.length>0) {
				
				
				//연도 데이터를 output data의 키로 설정
				outputKey.set(line[2].substring(0,4));
				
				// 상승마감
				if(jobType.equals("up")) {
					//상승마감을 판단해서 값을 추출(종가 - 시가)
					float resultValue = Float.parseFloat(line[6]) -
										Float.parseFloat(line[3]);
					if(resultValue > 0) { //상승마감 상태  - shuffle단으로 내보내기
						context.write(outputKey, one);
					}
					
				}else if(jobType.equals("down")) {
					//하락마감
					float resultValue = Float.parseFloat(line[6]) -
										Float.parseFloat(line[3]);
					
					if(resultValue < 0) { //하락마감 상태  - shuffle단으로 내보내기
						context.write(outputKey, one);
					}
				}//else if
			} //if(line)
		}
	}
}
```





**1-3) Driver 작성**

- StockOptionDriver.java

```java
package mapreduce.exam.option.stock;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class StockOptionDriver extends Configured implements Tool{
   @Override
   public int run(String[] optionlist) throws Exception {
      // 1. 사용자가 입력한 옵션을 처리하기 위해서 GenericOptionParser객체를 활용
      GenericOptionsParser optionParser = 
            new GenericOptionsParser(getConf(), optionlist);
      
      // optionlist에 포함된 -D옵션 이외에 사용자가 입력한 명령행 매개변수와 나머지 공통옵션들은
      // 따로 분리하는 작업을 해야 한다. jobType은 mapper로 전달할 옵션이고 inputpath와
      // outputpath는 일반 매개변수로 처리
      // GenericOptionsParser의 getRemainingArgs메소드를 호출하면 공통옵션과
      // 사용자가 입력한 매개변수를 따로 분리해서 리턴
      // otherArgs : 아래와 같은 옵션과 같이 입력하지 않은 명령형 매개변수
      // -D : 사용자가 입력한 값을 mapper에 전달
      // -fs 
      // -conf
      // -files...
      String[] otherArgs = optionParser.getRemainingArgs();

      // 2. job생성
      Job job = new Job(getConf(), "stockOption");
      job.setMapperClass(StockOptionMapper.class);
      job.setReducerClass(StockOptionReducer.class);
      job.setJarByClass(StockOptionDriver.class);
      
      // 3. input, output 포맷정의
      job.setInputFormatClass(TextInputFormat.class);
      job.setOutputFormatClass(TextOutputFormat.class);
      
      // 4. 리듀서의 출력데이터 key, value의 타입
      job.setOutputKeyClass(Text.class);
      job.setOutputValueClass(IntWritable.class);
      
      // 5. hdfs에 저장된 파일을 읽고 처리결과를 저장할 수 있도록 path정의
      FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
      FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
      job.waitForCompletion(true);
      return 0;
   }
   
   public static void main(String[] args) throws Exception {
      ToolRunner.run(new Configuration(), new StockOptionDriver(), args);
   }
}
```



- 이클립스로 jar 파일 하둡에 복사



- jar 파일 실행 
  - option : down

![image-20200928161906947](image/image-20200928161906947.png)



- up 결과 

![image-20200928161957915](image/image-20200928161957915.png)



- jar파일 실행
  - option : down

![image-20200928162101878](image/image-20200928162101878.png)



- down 결과

![image-20200928162218660](image/image-20200928162218660.png)



> mission

- 예제 2번 option 사용하여 도착 지연, 출발 지연 결과



- AirOptionMapper.java

```java
package mapreduce.exam.option.air;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class AirOptionMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	private final static IntWritable one = new IntWritable(1);
	private Text outputKey =new Text();
	
	private String jobType; //하둡 실행할 때 입력한 옵션값
	
	@Override
	protected void setup(Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		
		jobType = context.getConfiguration().get("jobType");
	}



	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		if(key.get()>0) {
			String[] line = value.toString().split(",");
			if(line!=null && line.length>0) {
				//도착지연 
				if(! line[14].equals("NA") && jobType.equals("Arr") && Integer.parseInt(line[14])>0) {
					//월 데이터를 output data의 키로 설정
					outputKey.set(line[1]+"월");
					context.write(outputKey, one);
					
				//출발지연	
				}else if(! line[15].equals("NA") &&jobType.equals("Dep") && Integer.parseInt(line[15])>0) {
					outputKey.set(line[1]+"월");
					context.write(outputKey, one);
				}
				
			}
		}
	}
}

```



- AirOptionReducer.java

```java
package mapreduce.exam.option.air;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class AirOptionReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
	
	private IntWritable resultVal = new IntWritable();
	
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,
			Reducer<Text, IntWritable, Text, IntWritable>.Context context) 
			throws IOException, InterruptedException {
		int sum = 0;
		
		//합산하는 작업
		for(IntWritable value : values) {
			sum = sum + value.get();
		}
		resultVal.set(sum); 
		context.write(key, resultVal);
	}

	
}

```



- AirOptionMapper.java

```java
package mapreduce.exam.option.air;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class AirOptionMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	private final static IntWritable one = new IntWritable(1);
	private Text outputKey =new Text();
	
	private String jobType; //하둡 실행할 때 입력한 옵션값
	
	@Override
	protected void setup(Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		
		jobType = context.getConfiguration().get("jobType");
	}



	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		if(key.get()>0) {
			String[] line = value.toString().split(",");
			if(line!=null && line.length>0) {
				//도착지연 
				if(! line[14].equals("NA") && jobType.equals("Arr") && Integer.parseInt(line[14])>0) {
					//월 데이터를 output data의 키로 설정
					outputKey.set(line[1]+"월");
					context.write(outputKey, one);
					
				//출발지연	
				}else if(! line[15].equals("NA") &&jobType.equals("Dep") && Integer.parseInt(line[15])>0) {
					outputKey.set(line[1]+"월");
					context.write(outputKey, one);
				}
				
			}
		}
	}
}

```



- option=Arr 수행

![image-20200928172514519](image/image-20200928172514519.png)



- 결과 

![image-20200928172443315](image/image-20200928172443315.png)



- option=Dep 수행

![image-20200928172343890](image/image-20200928172343890.png)



- 결과

![image-20200928172416968](image/image-20200928172416968.png)







