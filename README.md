# GSoC-firstTry-Note

笔记起于GSoC2020，因为没有事先Pull requests导致失去了竞争力

阅读Casbin，JCasbin源码时的笔记，因为对go语言还不太熟悉，几乎是硬着头皮看完Casbin的

+ JCasbin
  + config
    + Config: 用于读取配置的类
  + effect
    + Effect: 枚举常量, 没什么内容
    + Effector: 接口，用于比较用户条件与模型，没什么内容
    + DefaultEffector: 实现Effector，实现了一些默认的比较用户条件与模型的方法并返回Boolean
  + exception
    + CasbinAdapterException:  继承了Java的RuntimeException，没什么内容
    + CasbinConfigException: 继承了Java的RuntimeException, 没什么内容
    + CasbinMatcherException: 继承了Java的RuntimeException, 没什么内容
    + CasbinPolicyFileNotFoundException: 继承了Java的RuntimeException, 没什么内容
  + main
    + CoreEnforcer: 定义了Enforcer核心功能，包含了model，function，adapter等等的初始化方法和匹配得出结果的方法
    + Enforcer：Enforcer = ManagementEnforcer + RBAC API，有一些RBAC模型的添加删除查询方法
    + InternalEnforcer: InternalEnforcer = CoreEnforcer + Internal API, 有addPolicy，removePolicy方法
    + ManagementEnforcer: ManagementEnforcer = InternalEnforcer + Management API， 有一些获取conf文件中的模型属性的方法，获取object，subject，action，roles，policy
    + SyncedEnforcer：SyncedEnforcer = ManagementEnforcer + RBAC API，在Enforcer上添加了读写锁防止冲突
  + model
    + Assertion：Assertion represents an expression in a section of the model. For example: r = sub, obj, act.  这个类重要，里边有涉及RBAC模型的储存
    + FunctionMap: 将keyMatch，keyMatch2，regexMatch，ipMatch这些方法类（实现了AviatorFunction）装入Map中
    + Model：模型类，有各种增删改模型的方法
    + Policy： 策略类，有各种增删改查策略的方法
  + persist
    + file_adapter
      + FileAdapter：FileAdapter is the file adapter for Casbin，It can load policy from file or save policy to file
    + Adapter：Adapter is the interface for Casbin adapters
    + Helper：辅助FileAdapter
    + Watcher：Watcher is the interface for Casbin watchers
  + rbac
    + DefaultRoleManager：实现了RoleManager接口，见名知意
    + GroupRoleManager：继承了DefaultRoleManager，见名知意
    + RoleManager：接口
  + util
    + function: 这个文件夹下全是keyMatch，keyMatch2，regexMatch，ipMatch方法类
    + BuiltInFunctions：给keyMatch表达式给予支持
    + Util：工具类
+ Casbin（Filter表示过滤，可以理解为model和policy中的部分内容）
  + config
    + testdata
      + testini.ini：用于测试config的配置文件
    + config：用于配置，已经写好了读取配置文件的方法，一般不需要改动，还配备有测试方法，有不懂的地方可以用测试方法来检验
    + config_test：用于测试config
  + effect
    + effector：Effector is the interface for Casbin effectors，有个Effector接口
    + default_effector：DefaultEffector is default effector for Casbin，实现了MergeEffects方法：merges all matching results collected by the enforcer into a single decision
  + errors
    + rbac_errors：Global errors for rbac defined here
  + examples
    + 各种模型和策略文件，conf、csv
  + log
    + default_logger：DefaultLogger is the implementation for a Logger using golang log
    + log_util：log工具类
    + log_util_test：log_util测试
    + logger：Logger is the logging interface implementation
  + model
    + Assertion：Assertion represents an expression in a section of the model，For example: r = sub, obj, act，内有buildRoleLinks方法调用RoleManager进行角色绑定
      + Key:   r (p, e, g, m)
      + value:   sub,obj,act  (some(where (p_eft == allow)), r_sub == p_sub && r_obj == p_obj && r_act == p_act)
      + Tonkens:   [r_sub, r_obj, r_act]  ([p_sub p_obj p_act])    只有r，p有Tonkens
      + Policy:   二维数组\[][]  [alice] \[data1] \[read]
      + RM:   
    + function：FunctionMap represents the collection of Function，将keyMatch，keyMatch2等方法类（实现了ExpressionFunction）装入FunctionMap中
    + model：Model represents the whole access control model
      + key：r, p, e, g, m
      + AssertionNameMap：map[string]*Assertion
    + model_test：测试model
    + policy：BuildRoleLinks方法让RoleManager构建角色关系，还有各种关于policy的方法
  + persist
    + file-adapter：
      + adapter：Adapter is the file adapter for Casbin.  It can load policy from file or save policy to file
      + adapter_filtered：FilteredAdapter is the filtered file adapter for Casbin. It can load policy from file or save policy to file and supports loading of filtered policies.
      + adapter_mock：AdapterMock is the file adapter for Casbin. It can load policy from file or save policy to file.
    + adapter：Adapter is the interface for Casbin adapters.
    + adapter_filtered：FilteredAdapter is the interface for Casbin adapters supporting filtered policies.
    + persist_test：什么都没有写
    + watcher：Watcher is the interface for Casbin watchers
  + rbac
    + default-role-manager
      + role_manager：RoleManager provides a default implementation for the RoleManager interface, 这里的hasRole方法和代理冲突有关
      + role_manager_test：role_manager的测试
    + role_manager：RoleManager provides interface to define the operations for managing roles
  + util
    + builtin_operators：实现KeyMatch的模糊匹配
    + builtin_operators_test：测试builtin_operators
    + util：一些工具性方法
    + util_test：测试util
  + main
    + enforcer：Enforcer is the main interface for authorization enforcement and policy management.
    + enforcer：CachedEnforcer wraps Enforcer and provides decision cache
    + enforcer_cached：EnableCache determines whether to enable cache on Enforce(). When enableCache is enabled, cached result (true | false) will be returned for previous decisions.加入了缓存机制，节约了资源，不需要都进行查询
    + enforcer_cached_test：测试缓存机制
    + enforcer_interface：提供一个IEnforcer接口，IEnforcer is the API interface of Enforcer
    + enforcer_synced：SyncedEnforcer wraps Enforcer and provides synchronized access，加入了同步机制，有开启，停止定期加载policy的方法
    + enforcer_synced_test：测试enforcer_synced的同步机制
    + enforcer_test：测试enforcer
    + error_test：测试错误创建enforcer的报错功能
    + filter_test：测试enforce加载model和policy部分内容的功能
    + internal_api：
      + addPolicy -> addPolicy adds a rule to the current policy
      + removePolicy -> removePolicy removes a rule from the current policy
      + removeFilteredPolicy -> removeFilteredPolicy removes rules based on field filters from the current policy
    + management_api：
      + GetAllSubjects -> GetAllSubjects gets the list of subjects that show up in the current policy.
      + GetAllNamedSubjects -> GetAllNamedSubjects gets the list of subjects that show up in the current named policy.
      + GetAllObjects -> GetAllObjects gets the list of objects that show up in the current policy.
      + 等等
      + （其实就是调用enforce的方法，测试enforce的GetAllSubjects，GetAllObjects，GetPolicy等方法）
    + management_api_test：测试management_api的方法
    + rbac_api：RBAC模型角色用户相关的API
    + rbac_api_synced：在rbac_api的基础上增加了同步机制
    + rbac_api_with_domain：在rbac_api的基础上增加了前缀domain机制，domain is a prefix to the users (can be used for other purposes).
    + rbac_api_with_domain_synced：在rbac_api_with_domain的基础上，添加了同步机制

