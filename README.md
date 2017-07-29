# RELOGO-Zombie-Model
using repast simphony to simulate the zombie chasing human  
自学repast simphony，发觉网上尚未有中文指南，自己翻译了一下，原件为官方指南《RELOGO Getting started》，已翻译五分之四的内容，后续部分还在学习中，后续抽空补上。  

0.开始之前  
--
在使用repast symphony前我们需要确定已经正确安装了最新版本。下载及安装指南可以在官方网站上找到。repast symphony要求系统已安装Java 8。Java也可以在官网上自行下载。  
	
1.开始RELOGO  
-
现在让我们开始说明relogo。我们将会构建一个基于智能体的模型（僵尸追，人类逃）作为案例。我们并不是要吓你，只是为了尽可能清楚地解释每一个步骤。当这一章节结束时，我们将覆盖使用过程中的方方面面并且你将可以利用relogo开展你自己的探索。  
首先我们需要创建一个relogo project。这一目的通过点击顶部工具栏的“new relogo project” 按钮来达成。这一操作会使系统弹出新建项目引导窗口，并要求我们对新项目命名（其余选项我们暂且忽略）。在project name中键入Zombies，我们点击finish按钮使得智能引导的设置作用于项目。 
我们接下来所看到的就是zombies项目的文件夹，包括“str”子文件夹，“zombies.relogo”文件包以及相关的relogo文档，还有“shapes”文件夹，这些都是我们生成的项目文件。  
1.1创建人类与僵尸的智能体类型  
现在我们已经使所有的模型结构就位，我们可以开始指定zombie模型的细节了。首先，因为僵尸喜欢追逐人类，我们构建人类智能体的类型。单击选定“zombies.relogo”文件包，如果没有选定，点击工具栏上的“new turtle”按钮，此时系统会弹出新智能体引导窗口，让我们指定新智能体类型的名字。如果我们先前已经选定了“zombie.relogo”文件包，我们只需简单的在“name”栏中填上“human”并且点击finish按钮。现在新建的human智能体已经呈现在我们面前。  
1.2定义人类与僵尸的行为模式  
	构建模型的下一步是定义human和zombie的智能体行为模式。每个human智能体都会有一个step method（行走函数），看起来像这样：  
	def step(){
		def winner = minOneOf(neighbors()){
			count(zombiesOn(it))
		}
		face(winner)
		forward(1.5)
		if (infected()){
			infectionTime++
			if (infectionTime >= 5){
				hatchZombies(1){
					size = 2
				}
				die()
			}
		}
	}  
这里的思路是每当我们的模拟时间向前进，或者说前进一回合时，每个human智能体将会执行这个method。Human智能体会首先选择一个winner，简单来说，在周围的小块中选一个僵尸数目最少的。当选定winner以后，human智能体会face（面向）它，并向它移动1.5个单位，由此远离潜在的高僵尸风险区域。如果human智能体infected（感染）了，在被感染的5个回合后die（死亡）并hatchZombies（孵化出一只僵尸）。  

让我们简要回顾一下代码。一个relogo智能体拥有一些基本类型，这些变量可以不被新建而直接拿来使用，比如说minOneOf和neighbors。minOneOf变量设置一个“物件”，并定义一个决定最小值的数量。将鼠标悬在minOneOf变量上，可以看到相关信息。  

这个“物件”就是从neighbors变量返回的一系列小块，想象一个九宫格，人类站在正中间，周围的八个小块就是返回的数据。你会发觉代码中neighbors变量后面跟着一对空括号，这是因为基本类型在一些程序循环中会被当做method来查询，而空括号则充当基本类型的参数。即便一个method不带任何参数，就像上述的neighbors，我们仍然需要唤出method。就像上文所述，minOneOf带有两个参数，第二个函数是一串代码。为了表达清晰Groovy允许省去代码块附近的括号，如果这是method的最后一个参数，所以以下代码：  

	minOneOf(neighbors(),{
		count(zombiesOn(it))
		})  
		
可以被写成：  

	minOneOf(neighbors()){
		count(zombiesOn(it))
		}  

代码块会作为minOneOf的第一个参数被执行。  

当你在relogo中定义了一个智能体，就默认会有一些基本类型能运用到各种relogo实体上。ZombiesOn就是这样一个基本类型，当我们定义僵尸智能体时，它将参数作为补丁，并从补丁处返回僵尸智能体。“it”是groovy的另一个特点，对代码块而言这是一个默认参数，可以使我们将下列代码：  

	minOneOf(neighbors()){p->
		count(zombiesOn(p))
		}  
		
简化为：  
	
	minOneOf(neighbors()){
		count(zombiesOn(it))
		}  
	
基本类型“count”可以被用到任何一组对象当中并返回这组对象的数量。最后，关键词“def”在groovy中被用来定义一个变量或是method返回量的类型。大致上说，一个通配符表明不管它修饰什么，它就是“一些”类型，而不必进一步指定。所以说上面的代码就是给winner变量设计了一个僵尸数量最少的方向。
现在让我们看看完整的human智能体，它带有一个step method和它的智能体属性：

	class Human extends ReLogoTurtle {
		def infected = false
		def infectionTime = 0

		def step(){
			def winner = minOneOf(neighbors()){
				count(zombiesOn(it))
			}
			face(winner)
			forward(1.5)
			if (infected){
				infectionTime++
				if (infectionTime >= 5){
					hatchZombies(1){
						size = 2
					}
					die()
				}
			}
		}
	}

在class中我们已经定义了human智能体的method与属性，接下来我们看看zombie智能体，它看起来像是下面这样：

	class Zombie extends ReLogoTurtle {
		def step(){
			def winner = maxOneOf(neighbors()){
				count(humansOn(it))
			}
			face(winner)
			forward(0.5)
			if (count(humansHere()) > 0){
				label = "Brain!"
				infect(oneOf(humansHere()))
			}
			else {
				label = ""
			}
		}
		def infect(Human human){
			human.infected = true
		}
	}

不难发现，在step method中我们另外设置了infect作为辅助method，它有一个参数，引用自human智能体。让我们探索一下它是如何使用的。这里使用了基本类型“count”和“humansHere”，来看是否有任何human智能体在“Here”。如果有，我们就定义默认的智能体属性“label”为“brains!”，并且感染其中一个人类。为了感染这个人类，我们在一个时期后将human智能体的“infected”属性变为“true”。

1.3利用用户观察模块协调行为
现在我们已经定义了human和zombie两个智能体，接下来我们要做的就是定义模型的全部流程。这里我们使用“observer”来达成这个目的。当我们创建zombie project后，“UserObserver”是一个默认的观测类型，双击左侧的UserObserver.groovy，我们进入文件包浏览器。定义一个method的语法在relogo中是通用的，我们定义“setup”method如下：

	@Setup
		def setup(){
			clearAll()
			setDefaultShape(Human,"person")
			createHumans(numHumans){
				setxy(randomXcor(),randomYcor())
			}
			setDefaultShape(Zombie,"zombie")
			createZombies(numZombies){
				setxy(randomXcor(),randomYcor())
				size = 2
			}
		}
我们首先简要说明一下这段代码做了什么。利用初始化代码重置模拟是个好想法，所以我们“clearAll”世上存在的实体。然后我们定义人类与僵尸的默认形状，通过创造不同的数值的基本类型。我们也将创造出的智能体随机散布到世界各地，并将僵尸的大小设置为2，使其看起来更显著。

更深入一些说，@setup是一个注释，这向系统表明，这个method应当在系统开始运行时就运行。setDefaultShape是一个基本类型，有两个参数。第一个参数是智能体类型，第二个是定义智能体形状的字符串。想了解那些形状是可以用的话，就点击左侧“shape”文件夹，就可以看到relogo project里所有的默认形状。如果只简单的设置了第一个参数（智能体名字），那么智能体会设置成默认形状。

聪明的读者可能已经发现代码中的numHumans和numZombies了，他们可以被当作UserObserver的属性，我们选择把这些东西作为仿真中可滑动的图形元素，relogo编辑器会自动完成这些。键入createHu后会在代码下方弹出潜在的代码比如“createHumans” method，此时按空格确认即可自动完成剩余的代码。与之类似的还有创建类型后提示参等。

接下来我们需要定义模拟的步骤，也就是说当时间望前走时，僵尸和人类会共同发生什么。我们通过@go method来完成：

	@Go
		def go(){
			ask (zombies()){
				step()
			}
			ask (humans()){
				step()
			}
		
正如所见，我们已经完成了定义智能体行为中最难的工作。“@go”这个注释告诉系统，系统每运行一个单位时间，就要重复运行一次@go method。检测器所做的就是让human以及zombie智能体类执行他们的step method——这个功能通过“ask”基本类型来完成。一个relogo实体可以ask另一个或是另一组实体做一些事情，通过指定ask基本类型后的对象。在这种情况下，zombies和humans基本类型会各自返回一组僵尸和人类智能体。

我们现在的代码看起来像是这样：

	class UserObserver extends ReLogoObserver{
		@Setup
		def setup(){
			clearAll()
			setDefaultShape(Human,"person")
			createHumans(numHumans){
				setxy(randomXcor(),randomYcor())
			}
			setDefaultShape(Zombie,"zombie")
			createZombies(numZombies){
				setxy(randomXcor(),randomYcor())
				size = 2
			}
		}
		@Go
		def go(){
			ask (zombies()){
				step()
			}
			ask (humans()){
				step()
			}
		}
	}

1.4创建图形控制与显示组件

我们已经制定了智能体行为模式与模型所有流程，现在我们要加入一些图形控制组件。于此有关的文件是“UserGlobalAndPanelFactory.groovy”。打开这个文件我们看到一些可能有帮助的可用组件。为了完成这个列表，我们通过标签创建了滑动组件（就是上面所说的slider模块），以此控制“UserObserver”中的“numHumans”以及“numZombies”两个变量，代码如下：

		addSliderWL("numHumans","Number of Humans",1,1,100,50)
		addSliderWL("numZombies","Number of Zombies",1,1,10,5)
在模拟时，我们希望知道当前还剩余多少人类，于是我们用到了monitor监视器，代码如下：

		addMonitorWL("remainingHumans","Remaining Humans",5)
		
monitor第一个参数是“UserObserver”中一个method的名字，第二个参数是显示在monitor的标签，第三个参数定义多久更新一次monitor。5意味着每5回合更新一次。为了知道剩余人类的数量，我们对“UserObserver”的代码做了一点改进。
现在整个代码看起来像是这样：

	class UserObserver extends ReLogoObserver{
		@Setup
		def setup(){
			clearAll()
			setDefaultShape(Human,"person")
			createHumans(numHumans){
				setxy(randomXcor(),randomYcor())
			}
			setDefaultShape(Zombie,"zombie")
			createZombies(numZombies){
				setxy(randomXcor(),randomYcor())
				size = 2
			}
		}
		@Go
		def go(){
			ask (zombies()){
				step()
			}
			ask (humans()){
				step()
			}
		}
		def remainingHumans(){
			count(humans())
			}
	}

1.5运行僵尸模型

下一步就是看看我们制作至今的模型运行后究竟会发生什么。点击左上方工具栏内的小三角形、绿色的开始按钮，并在下拉菜单中选择“zombies model”选项，这会启动repast simphony runtime程序。接下来点击初始化按钮使得模型的缓存清空。

左侧的用户操作界面有我们设置的图形组件。点击step按钮，模型会运行一个单位时间。这时候我们可以反复点击step按钮观察结果，也可以点击play按钮让模型自动运行，直到我们按下pause或者stop按钮。无论何时我们想重置模型，只需要点击reset按钮即可。如果我们想要改进或是在模型中添加代码，则必须关闭repast simphony runtime程序，并在修正完毕后重新启动。

1.6为僵尸模型添加连接

在这一小节我们将介绍如何为模型添加连接。我们新建名为“Infection”的连接类型，以此追踪究竟是哪个僵尸智能体感染了哪个人类智能体。这个操作和前文创建human以及 zombie智能体的操作相似——单击选择“zombies.relogo”文件包，并点击工具栏的“new link”按钮。这时系统会弹出new link导引，让我们可以指定我们link的名字。我们在name栏里填入“Infection”并点击finish按钮。
现在我们稍稍改进一下zombie模型的step method，看起来像是这样：

	def step(){
			def winner = maxOneOf(neighbors()){
				count(humansOn(it))
			}
			face(winner)
			forward(0.5)
			if (count(humansHere()) > 0){
				label = "Brain!"
				infect(oneOf(humansHere()))
				def infectee = oneOf(humansHere())
				infect(infectee)
				createInfectionTo(infectee)
			}
			else {
				label = ""
			}
		}
	
这些修改达成了这样的效果——如果僵尸找到了一个人类，它除了会感染人类，还会在二者之间创造一条Infection连接。当感染者死亡后，Infection连接就自动移除，因而新孵化的僵尸将不会有任何连接。当新建连接类型时我们需要通过定义新的method来得到新连接的数量，这和当我们当初定义智能体类型时指定method是差不多的。“createInfectionTo”就是这样一个method。

我们也想要把孕育僵尸的周期加进模型玩玩儿，就比如说人类从死亡再变成僵尸的这段时间，可以让我们看到一个在感染者死亡前的感染网络。我们把gestationPeriod变量作为一个slide模块加入代码，看起来像是这样：

	public void addGlobalsAndPanelComponents(){
			addSliderWL("numHumans","Number of Humans",1,1,100,50)
			addSliderWL("numZombies","Number of Zombies",1,1,10,5)
			addSliderWL("gestationPeriod", "Gestation",5,1,30,5)
			addMonitorWL("remainingHumans","Remaining Humans",5)
		}
	
并且我们还稍稍改进了一下human智能体的代码：

	class Human extends ReLogoTurtle {
		def infected = false
		def infectionTime = 0

		def step(){
			def winner = minOneOf(neighbors()){
				count(zombiesOn(it))
			}
			face(winner)
			forward(1.5)
			if (infected){
				infectionTime++
				if (infectionTime >= gestationPeriod){
					hatchZombies(1){
						size = 2
					}
					die()
				}
			}
		}
	}

现在运行我们的模型，当我们增加孕育周期时可以看到更大的感染连接网络形成了。

1.7数据组，输出以及外部插件
#
现在我们已经运行过模型，我们将继续探索如何输出数据组，甚至更进一步，使用一些额外的扩展插件。现在我们启动模型，启动后我们将目光移到左侧的scenario tree并照着下面的步骤一步一步执行：  

（1）右键点击data sets label 并选择add datasets；

（2）在data sets编辑器中的data set ID一栏中填入Agent Counts，并在data set type的下拉菜单里选择Aggregate；

（3）下面的步骤我们将指定standard source来存放我们的数据组，这一步不用修改任何东西，保持默认；

（4）接下来，选定method data source，点击add按钮增加一列。在source name一栏键入“Remaining Humans”，在agent type中选择Human，并在aggregate operation中选择count；

（5）再次点击add，这次在source name一栏键入“Remaining Zombies”，在agent type中选择Zombie, 并在aggregate operation中选择count。

点击next，再点击finish，通过左上角的保存按钮保存设置。

现在我们已经创建了想要输出的数据组并设置好了一切，下面我们要将运行数据输出到sink文件中。我们需要完成以下几个步骤：

（1）右键点击scenario tree中的text sinks（往下滑动一下就能看到），并点击add file sink；  

（2）随便输入什么作为数据集的名字；  

（3）在第一栏里点击tick并点击中间的绿色按钮；  

（4）重复操作（3）直至所有项目都挪到右边那一栏；  

（5）点击next，点击finish（未提到的全部保持默认）。  

我们已经选择所有希望输出的数据项目，记得保存设置。  

现在初始化运行时间，并通过step或者play按钮运行模型，如果我们发现左侧工作区没有任何改变，就选定zombies project并右键刷新（refresh）它，这时候就能看见新建的数据文件了。双击这个文件我们可以打开它。  

我们还可以玩玩儿外部插件，先启动模型，初始化后运行模型，这时我们将注意力转向右上方——拓展工具栏。通过spreadsheets插件我们可以轻松将结果输出到Excel中。点击spreadsheets按钮，如果你看见license界面，点next。在spreadsheets的主界面中，选定你电脑上Excel的位置，点next然后你会看到sink文件已经被选定。点击finish，继而Excel就会启动并在工作表里展示你的数据。  

我们建议使用者自己探索插件的使用方法。  
