	
	//厂商名字
	def changshang ='weimi'
	//要备份的apk的目录
	def apkDir ='../signedApk'
	//要备份的版本号
	def files = file(apkDir).listFiles()
	//备份的路径
	def backUpPath ='F:/ireader_work_changshang/backUpSdk/'+changshang+'/'+ getVersion()
	
	def version =getVersion();
	
	//排序，取最后生成的
	Arrays.sort(files,new CompratorByLastModified())
	println files.getAt(0).name  //plug.zip
	println files.getAt(1).name  //mapping.txt
	println files.getAt(2).name  //apk
	
	task backUpSdk(type: Copy){
	
	    from(apkDir){
	        include files.getAt(0).name
	        include files.getAt(1).name
	        include files.getAt(2).name
	    }
	
	    //备份sdk
	    from('../IreaderPlugSdk/build/intermediates/bundles/release'){
	        include 'classes.jar'
	    }
	
	
	    //重命名
	    rename 'classes.jar', 'IreaderPlugSdk.jar'
	
	    //备份mapping
	    from('../IreaderPlugSdk/build/outputs/mapping/release'){
	        include 'mapping.txt'
	    }
	
	    rename 'mapping.txt','IreaderPlugSdk-mapping.txt'
	
	
	    println backUpPath
	    into(backUpPath)
	}
	
	task GenZip(type: Zip,dependsOn:backUpSdk){
	    def apkPath;
	
	    if (files.getAt(0).name.endsWith('zip')){
	        apkPath=files.getAt(0).name;
	    }else if (files.getAt(1).name.endsWith('zip')){
	        apkPath=files.getAt(1).name;
	    }else {
	        apkPath=files.getAt(2).name;
	    }
	    println(apkPath)
	
	    from apkDir
	    include apkPath
	
	
	    from '../IreaderPlugSdk/build/intermediates/bundles/release'
	    include 'classes.jar'
	    //重命名
	    rename 'classes.jar', 'IreaderPlugSdk.jar'
	
	    //压缩文件存放的目录
	    destinationDir = file(backUpPath)
	    //压缩文件的名字
	    archiveName =  changshang+'_'+version+'_'+getDate()+'.zip'
	}
	
	
	//根据文件修改时间进行比较  最晚修改的放在前面
	class CompratorByLastModified implements Comparator<File> {
	
	    public int compare(File f1, File f2) {
	        long diff = f1.lastModified() - f2.lastModified();
	        if (diff > 0) {
	            return -1;
	        } else if (diff == 0) {
	            return 0;
	        } else {
	            return 1;
	        }
	    }
	}
	
	def getVersion(){
	    def mainfest =new XmlSlurper().parse(file('src/main/AndroidManifest.xml'))
	    return mainfest.getProperty('@android:versionName')
	}
	
	import java.text.SimpleDateFormat
	def getDate(){
	//    return new Date().format( 'yyyyMMdd' )
	    Date data =new Date();
	    SimpleDateFormat simpleDateFormat =new SimpleDateFormat("yyyyMMdd");
	    return simpleDateFormat.format(data);
	}