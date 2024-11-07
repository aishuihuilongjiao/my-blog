#### 接口设计

##### getInfo接口

是

| 请求方式 | 请求地址 | 请求头          |
| -------- | -------- | --------------- |
| GET      | /getInfo | 需要token请求头 |

请求参数：

无

响应格式:

如果用户id为1代表管理员，roles 中只需要有admin，permissions中需要有所有菜单类型为C或者F的，状态为正常的，未被删除的权限

~~~~json
{
	"code":200,
	"data":{
		"permissions":[
			"system:user:list",
            "system:role:list",
			"system:menu:list",
			"system:user:query",
			"system:user:add"
            //此次省略1000字
		],
		"roles":[
			"admin"
		],
		"user":{
			"avatar":"http://r7yxkqloa.bkt.clouddn.com/2022/03/05/75fd15587811443a9a9a771f24da458d.png",
			"email":"23412332@qq.com",
			"id":1,
			"nickName":"sg3334",
			"sex":"1"
		}
	},
	"msg":"操作成功"
}
~~~~


RoleServiceImpl  selectRoleKeyByUserId方法

~~~~java
@Service("menuService")
public class MenuServiceImpl extends ServiceImpl<MenuMapper, Menu> implements MenuService {

    @Override
    public List<String> selectPermsByUserId(Long id) {
        //如果是管理员，返回所有的权限
        if(id == 1L){
            LambdaQueryWrapper<Menu> wrapper = new LambdaQueryWrapper<>();
            wrapper.in(Menu::getMenuType,SystemConstants.MENU,SystemConstants.BUTTON);
            wrapper.eq(Menu::getStatus,SystemConstants.STATUS_NORMAL);
            List<Menu> menus = list(wrapper);
            List<String> perms = menus.stream()
                    .map(Menu::getPerms)
                    .collect(Collectors.toList());
            return perms;
        }
        return getBaseMapper().selectPermsByUserId(id);
    }
}
~~~~

MenuMapper 

~~~~java
public interface MenuMapper extends BaseMapper<Menu> {

    List<String> selectPermsByUserId(Long userId);
}

~~~~
因为sql语句涉及多表，因此采用xml的形式来写
~~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.sangeng.mapper.MenuMapper">

    <select id="selectPermsByUserId" resultType="java.lang.String">
        SELECT
            DISTINCT m.perms
        FROM
            `sys_user_role` ur
            LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
            LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
        WHERE
            ur.`user_id` = #{userId} AND
            m.`menu_type` IN ('C','F') AND
            m.`status` = 0 AND
            m.`del_flag` = 0
    </select>
</mapper>
~~~~





MenuServiceImpl  selectPermsByUserId方法

~~~~java
@Service("roleService")
public class RoleServiceImpl extends ServiceImpl<RoleMapper, Role> implements RoleService {

    @Override
    public List<String> selectRoleKeyByUserId(Long id) {
        //判断是否是管理员 如果是返回集合中只需要有admin
        if(id == 1L){
            List<String> roleKeys = new ArrayList<>();
            roleKeys.add("admin");
            return roleKeys;
        }
        //否则查询用户所具有的角色信息
        return getBaseMapper().selectRoleKeyByUserId(id);
    }
}
~~~~

~~~~java
public interface RoleMapper extends BaseMapper<Role> {

    List<String> selectRoleKeyByUserId(Long userId);
}

~~~~

~~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.sangeng.mapper.RoleMapper">
    <select id="selectRoleKeyByUserId" resultType="java.lang.String">
        SELECT
            r.`role_key`
        FROM
            `sys_user_role` ur
            LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
        WHERE
            ur.`user_id` = #{userId} AND
            r.`status` = 0 AND
            r.`del_flag` = 0
    </select>
</mapper>
~~~~
