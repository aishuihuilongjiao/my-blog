
##### getRouters接口

| 请求方式 | 请求地址    | 请求头          |
| -------- | ----------- | --------------- |
| GET      | /getRouters | 需要token请求头 |

请求参数：

无

响应格式:

​	前端为了实现动态路由的效果，需要后端有接口能返回用户所能访问的菜单数据。

​	注意：**返回的菜单数据需要体现父子菜单的层级关系**

​	如果用户id为1代表管理员，menus中需要有所有菜单类型为C或者M的，状态为正常的，未被删除的权限

​	数据格式如下：

~~~~json
{
	"code":200,
	"data":{
		"menus":[
			{
				"children":[],
				"component":"content/article/write/index",
				"createTime":"2022-01-08 11:39:58",
				"icon":"build",
				"id":2023,
				"menuName":"写博文",
				"menuType":"C",
				"orderNum":"0",
				"parentId":0,
				"path":"write",
				"perms":"content:article:writer",
				"status":"0",
				"visible":"0"
			},
			{
				"children":[
					{
						"children":[],
						"component":"system/user/index",
						"createTime":"2021-11-12 18:46:19",
						"icon":"user",
						"id":100,
						"menuName":"用户管理",
						"menuType":"C",
						"orderNum":"1",
						"parentId":1,
						"path":"user",
						"perms":"system:user:list",
						"status":"0",
						"visible":"0"
					},
					{
						"children":[],
						"component":"system/role/index",
						"createTime":"2021-11-12 18:46:19",
						"icon":"peoples",
						"id":101,
						"menuName":"角色管理",
						"menuType":"C",
						"orderNum":"2",
						"parentId":1,
						"path":"role",
						"perms":"system:role:list",
						"status":"0",
						"visible":"0"
					},
					{
						"children":[],
						"component":"system/menu/index",
						"createTime":"2021-11-12 18:46:19",
						"icon":"tree-table",
						"id":102,
						"menuName":"菜单管理",
						"menuType":"C",
						"orderNum":"3",
						"parentId":1,
						"path":"menu",
						"perms":"system:menu:list",
						"status":"0",
						"visible":"0"
					}
				],
				"createTime":"2021-11-12 18:46:19",
				"icon":"system",
				"id":1,
				"menuName":"系统管理",
				"menuType":"M",
				"orderNum":"1",
				"parentId":0,
				"path":"system",
				"perms":"",
				"status":"0",
				"visible":"0"
			}
		]
	},
	"msg":"操作成功"
}
~~~~
service，获取菜单权利，并且返回
~~~~

MenuServiceImpl

~~~~java
@Override
    public List<Menu> selectRouterMenuTreeByUserId(Long userId) {
        MenuMapper menuMapper = getBaseMapper();
        List<Menu> menus = null;
        //判断是否是管理员
        if(SecurityUtils.isAdmin()){
            //如果是 获取所有符合要求的Menu
            menus = menuMapper.selectAllRouterMenu();
        }else{
            //否则  获取当前用户所具有的Menu
            menus = menuMapper.selectRouterMenuTreeByUserId(userId);
        }

        //构建tree
        //先找出第一层的菜单  然后去找他们的子菜单设置到children属性中
        List<Menu> menuTree = builderMenuTree(menus,0L);
        return menuTree;
    }

    private List<Menu> builderMenuTree(List<Menu> menus, Long parentId) {
        List<Menu> menuTree = menus.stream()
                .filter(menu -> menu.getParentId().equals(parentId))
                .map(menu -> menu.setChildren(getChildren(menu, menus)))
                .collect(Collectors.toList());
        return menuTree;
    }

    /**
     * 获取存入参数的 子Menu集合
     * @param menu
     * @param menus
     * @return
     */
    private List<Menu> getChildren(Menu menu, List<Menu> menus) {
        List<Menu> childrenList = menus.stream()
                .filter(m -> m.getParentId().equals(menu.getId()))
                .map(m->m.setChildren(getChildren(m,menus)))
                .collect(Collectors.toList());
        return childrenList;
    }
~~~~

MenuMapper.java

~~~~java
    List<Menu> selectAllRouterMenu();

    List<Menu> selectRouterMenuTreeByUserId(Long userId);
~~~~


同样因为sql语句比较复杂，自己在xml写

MenuMapper.xml

~~~~xml
 <select id="selectAllRouterMenu" resultType="com.sangeng.domain.entity.Menu">
        SELECT
          DISTINCT m.id, m.parent_id, m.menu_name, m.path, m.component, m.visible, m.status, IFNULL(m.perms,'') AS perms, m.is_frame,  m.menu_type, m.icon, m.order_num, m.create_time
        FROM
            `sys_menu` m
        WHERE
            m.`menu_type` IN ('C','M') AND
            m.`status` = 0 AND
            m.`del_flag` = 0
        ORDER BY
            m.parent_id,m.order_num
    </select>
    <select id="selectRouterMenuTreeByUserId" resultType="com.sangeng.domain.entity.Menu">
        SELECT
          DISTINCT m.id, m.parent_id, m.menu_name, m.path, m.component, m.visible, m.status, IFNULL(m.perms,'') AS perms, m.is_frame,  m.menu_type, m.icon, m.order_num, m.create_time
        FROM
            `sys_user_role` ur
            LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
            LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
        WHERE
            ur.`user_id` = #{userId} AND
            m.`menu_type` IN ('C','M') AND
            m.`status` = 0 AND
            m.`del_flag` = 0
        ORDER BY
            m.parent_id,m.order_num
    </select>
~~~~



查询的列：

SELECT DISTINCT m.id, m.parent_id, m.menu_name, m.path, m.component, m.visible, m.status, IFNULL(m.perms,'') AS perms, m.is_frame,  m.menu_type, m.icon, m.order_num, m.create_time

注意需要按照parent_id和order_num排序
 