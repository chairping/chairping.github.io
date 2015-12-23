---
layout:    post
title:     "Laravel Database学习笔记"
subtitle:  "基于Eloquent 4.2"
date:       2015-12-22 17:50
---

> [官方文档](http://www.golaravel.com/laravel/docs/4.2/database/, "传送门")

#### 数据库设置
{% highlight php startinline %}  
use \Illuminate\Database\Capsule\Manager as DB;
$db = new DB();

// 添加连接数据库配置信息, 默认标记为 'default'
$db->addConnection([
    'driver'    => 'mysql',
    'host'      => '127.0.0.1',
    'database'  => 'eloquent_learn',
    'username'  => 'root',
    'password'  => '111111',
    'charset'   => 'utf8',
    'collation' => 'utf8_general_ci',
    'prefix'    => ''
]); 
// 添加第二个连接数据库配置信息, 标记为 'connection'
$db->addConnection([
    'driver'    => 'mysql',
    'host'      => '127.0.0.1',
    'database'  => 'eloquent_learn',
    'username'  => 'root',
    'password'  => '111111',
    'charset'   => 'utf8',
    'collation' => 'utf8_general_ci',
    'prefix'    => ''
], 'connection');

$dbmanage->setAsGlobal(); //步骤1： 设置DBManager 实例为全局 
class_alias('Illuminate\Database\Capsule\Manager', 'DB');  //  步骤2：设置别名
// 步骤1， 步骤2 使得我们应用能像laravel一样 使用DB类名来操作数据库

// Setup the Eloquent ORM... (optional; unless you've used setEventDispatcher())
$dbmanage->bootEloquent();
{% endhighlight %}

#### 关闭sql日志记录（4.2默认情况下 开启日志记录， 只能手动关闭）：
{% highlight php startinline %}  
// 1. 关闭默认数据库的查询日志  默认数据库设置：
DB::connection()->disableQueryLog(); 
//or  
$db->connection()->disableQueryLog();

// 2. 关闭指定数据库查询日志
DB::connection('user')->disableQueryLog(); 
//or  
$db->connection('user')->disableQueryLog();

// 3. 关闭所有的数据查询日志
$connectList = $db->getContainer()['config']['database.connections'];
foreach ($connectList as $k => $v) {
    $db->getConnection($k)->disableQueryLog();
}
{% endhighlight %}

#### 基本使用：
{% highlight php startinline %}  
DB::select('select * from table where id = ?', array(1));
DB::insert('insert into table (field1, field2) values (?, ?)', array(1, 'Dayle'));
DB::getQueryLog();
{% endhighlight %}


#### Collection数据集（ get()或all()等方法返回的数据集合）：

 查找多条语句的时候 orm 会把结果封装到 Collection 类中， Collection 把常用的 `array 函数操作` 和 `自定义的函数操作` 封装到该类中， 以面向对象的形式操作数据和优雅的方法命名.

{% highlight php startinline %}  
class City extends \Illuminate\Database\Eloquent\Model {
    protected $table = 'city';
}

$citysCollection = City::all(); // 返回的是 Collection 对象 作用：对数据灵活操作

$citysCollection instanceof \Illuminate\Database\Eloquent\Collection && var_dump('true');
// 数据全部保存到 $citysCollection的items属性中

$citysCollection->isEmpty();      // 判断数据是否为空
$list = $citysCollection->all();  // 返回所有数据

$citysCollection->put('hahha', 'ahhahahahhaha'); // 相当于 $items['hahha']  ='ahhahahahahah';

$data = $citysCollection->first(); // 获取第一条数据
$data = $citysCollection->last();  // 获取最后一条数据
$data = $citysCollection->shift(); // return array_shift($this->items);
$citysCollection->push();          // array_unshift($this->items, $value);
$data = $citysCollection->pop();   // return array_pop($this->items);
$citysCollection->forget('hahha'); //unset($this->items[$key]);
$citysCollection->count();

$newObj = $citysCollection->reverse();

//  转化为单一的值 output: string 'initial 福州 厦门 莆田 三明 泉州 漳州 南平 龙岩 宁德' (length=70)
$reduce = $citysCollection->reduce(function($names, $row){
    return $names .' ' . $row->name;
}, 'initial');

$each = $citysCollection->each(function($row){
    if ($row->province_id == 12) {
        return $row;
    }
});

// 相当于 array_map();
$map = $citysCollection->map(function($row, $id){
    return $id . '-' . $row->name;
});
var_dump($map->toArray());
//array (size=9)
//  0 => string '0-福州' (length=8)
//  1 => string '1-厦门' (length=8)
//  2 => string '2-莆田' (length=8)
//  3 => string '3-三明' (length=8)
//  4 => string '4-泉州' (length=8)
//  5 => string '5-漳州' (length=8)
//  6 => string '6-南平' (length=8)
//  7 => string '7-龙岩' (length=8)
//  8 => string '8-宁德' (length=8)

$filter = $citysCollection->filter(function($row){
    return ($row->id & 1);
});
var_dump(array_column($filter->toArray(), 'id'));
//array (size=5)
//  0 => int 1
//  1 => int 3
//  2 => int 5
//  3 => int 7
//  4 => int 9

$aa = $citysCollection->fetch('name');
//array (size=9)
//  0 => string '福州' (length=6)
//  1 => string '厦门' (length=6)
//  2 => string '莆田' (length=6)
//  3 => string '三明' (length=6)
//  4 => string '泉州' (length=6)
//  5 => string '漳州' (length=6)
//  6 => string '南平' (length=6)
//  7 => string '龙岩' (length=6)
//  8 => string '宁德' (length=6)

$data = $citysCollection->lists('name');
//array (size=9)
//  0 => string '福州' (length=6)
//  1 => string '厦门' (length=6)
//  2 => string '莆田' (length=6)
//  3 => string '三明' (length=6)
//  4 => string '泉州' (length=6)
//  5 => string '漳州' (length=6)
//  6 => string '南平' (length=6)
//  7 => string '龙岩' (length=6)
//  8 => string '宁德' (length=6)

$data = $citysCollection->lists('name', 'city_id');
{% endhighlight %}

#### hasMany 与 预加载查询

{% highlight php startinline %}  
use Illuminate\Database\Eloquent\Model;

class Province extends  Model
{
    protected $table = 'province';
    public $timestamps = false;

    public function citys() {
        return $this->hasMany(City::class, 'province_id', 'id');
    }
}

class City extends  Model
{
    protected $table = 'city';
    public $timestamps = false;

    public function districts() {
        return $this->hasMany(District::class, 'city_id', 'id');
    }
}

class District extends  Model
{
    protected $table = 'district';
    public $timestamps = false;
}

//laravel对预加载的说明： 我们可以使用预载入大量减少查询次数。使用 with 方法指定想要预载入的关联对象：
foreach (Book::with('author')->get() as $book)
{
    echo $book->author->name;
}

//现在，上面的循环总共只会执行两次查询：
select * from books
select * from authors where id in (1, 2, 3, 4, 5, ...)
//使用预载入可以大大提高程序的性能。

//当然，也可以同时载入多种关联：
$books = Book::with('author', 'publisher')->get();
//甚至可以预载入巢状关联：
$books = Book::with('author.contacts')->get();
//上面的例子中， author 关联会被预载入， author 的 contacts 关联也会被预载入。


// 方法一
// 获取省份加载城市列表
$info = Province::with('citys')->get();
// 获取省份列表并加载城市列表， 同时城市列表加载区域列表
$info = Province::with('citys.districts')->get();
OUTPUT:
    array (
      0 => 
      array (
        'id' => '11',
        'name' => '北京',
        'citys' => 
            array (
              0 => 
                  array (
                    'id' => '439',
                    'name' => '北京',
                    'full_name' => '北京市',
                    'province_id' => '11',
                    'districts' => 
                        array (
                          0 => 
                          array (
                            'id' => '142',
                            'name' => '东城',
                            'province_id' => '11',
                            'city_id' => '439',
                          ),
                          ......
                    ),
              ),
              .............  
        ),
      ),
      .............  
    )

    // 方法二(可根据条件查询)
        $info = Province::with(
                    array(
                        'citys' => function($query){
                            $query->with('districts')->select('id', 'name', 'province_id'); // 这里必须要有外键·province_id·
                        })
                )
                ->limit('2')
                ->select('id','name')
                ->get();

{% endhighlight %}

#### 关联模型
{% highlight php startinline %}  
class City extends \Illuminate\Database\Eloquent\Model {
    protected $table = 'city';

    public function province()
    {
        $this->belongsTo(Province::class);
    }

}
class Province extends \Illuminate\Database\Eloquent\Model {
    protected $table = 'province';

    public function cities()
    {
        return $this->hasMany(City::class);
    }
}

// 获取id为12的省份信息
$province = Province::find(12);
// 例1：获取相关联的城市信息
$province->cities; // city的数据保存在province里面
// 例2：获取城市信息
$cities = $province->cities()->whereIn('id', array(1,3,4))->select('id', 'name')->get();
// 例3：获取城市信息
$citys = Province::find(12)->cities;

// 预载入
$provinces = Province::with('cities')->get();
//select * from province
//select * from city where province_id in (11, 12, 13, 14, 15, ...)

// 加条件
$provinces = Province::with(array('cities'=> function($query){
    $query->where('id', '>', 10);
}))->get();



$city = City::find(1)                   // 获取id为1的城市信息
$city->full_name = '福州测试1';          // 修改该城市的全称
$city->province->full_spell = 'test1'; // 修改该城市对应省份的全拼

// 更新数据 方法一
$city->save();   // 保存城市数据
$city->province->save(); // 保存省份数据
// 更新数据 方法二（该方法自动执行上面两步操作）
$city->push();
{% endhighlight %}

####  增改汇总
{% highlight php startinline %}  
$city = City::firstOrNew(array('province_id' => 100000));
if ($city->exists === true) {
    throw new \Exception("xxxxx");
}
$city->name= 'jajjaja';
$city->save();


// 以下方法没有白名单和 黑名单  所以必须确保传进去的key 在数据库中有存在 否则会出错
$status = City::insertGetId($data);  // 返回新增后的id
$status = City::insert($data); // 返回bool
$status = CarBrand::where('id', 3)->update($data);

//====================  相关操作  ========================//
// 没有记录 则插入新记录
City::firstOrCreate();
// 没有记录 则创建一个对象并返回
City::firstOrNew();
// 更新或插入操作
City::updateOrCreate();
City::findOrNew();
{% endhighlight %}

#### 多对多
{% highlight sql startinline %}  
TABLE `group_user` (
`id` ,
`user_id`,
`group_id`,
`create_time,
);

TABLE `groups` (
`id` ,
`create_time`,
`update_time`,
);

TABLE `users` (
`id` ,
`telephone` ,
`email` ,
);
{% endhighlight %}
{% highlight php startinline %}  
class Group extends Model {
    /**
     * @desc   多对多查询
     */
    public function users() {
        return $this->belongsToMany(User::class, 'group_user', 'group_id', 'user_id');
    }
}

//一、
$group = Group::find(1);

// 1. 建立 user 表 和 group表的数据关联
$userIds = User::get()->fetch('id')->toArray();
//在group_user表中插入相应的 group.id  user.id
$group->users()->attach($userids, ['create_time' => time()]); // 无返回值

// 2. 解除 user 表 和 group表的数据关联
$userId = 2;
$status = $group->users()->detach($userId);

// 3. 同步  user 表 和 group表的数据关联
$result = $group->users()->sync($userids);
$id = 1;
$result = $group->users()->sync([
    $id => ['create_time' => time()]
]);


//二、
$group = Group::all()->last();

// 1. 查出所有相关联的用户数据
$group->users;
var_dump(group->toArray())
// output :
   array (size=18)
     'id' => int 112
     'users' =>
       array (size=311)
         0 =>
           array (size=28)
             'id' => int 1
             'telephone' => string '1111111111' (length=11)
             'pivot' =>
               array (size=2)
                     'group_id' => int 112
                      'user_id' => int 1
         1 =>
        ........

// 2. 根据条件查询
$users = $group->users()->select('nickname', 'email')->get();
 var_dump($users->toArray());
//   0 =>
       array (size=3)
         'email' => string '' (length=0)
         'pivot' =>
           array (size=2)
             'group_id' => int 112
             'user_id' => int 1
     .........


$group->users() instanceof Illuminate\Database\Eloquent\Relations\BelongsToMany

//三
$normalGroup = $this->model->normal()->whereHas('groupUser', function($q) use($uid){
        $q->where('user_id', $uid);
    })->get()->toArray();
    
$group->users()->select('users.photo', 'group_user.group_id', 'group_user.user_id')->chunk(100, function($uesrs) use (&$result){
        $result = array_merge($result, $uesrs->toArray());
    });
 
{% endhighlight %}

#### 远程关系
{% highlight php startinline %}  
use Illuminate\Database\Eloquent\Model;
class Province extends  Model
{
    protected $table = 'province';
    // 1.   province.id -----> city.province_id  
    // 2.   city.id     -----> district.city_id
    // final get district info
    public function districts() {
        return $this->hasManyThrough(District::class, City::class, 'province_id', 'city_id');
    }
}
class City extends  Model
{
    protected $table = 'city';
}
class District extends  Model
{
    protected $table = 'district';
}

$province = Province::find(12);
$a = $province->districts;
var_dump($a->toArray());
{% endhighlight %}

#### 白名单 
{% highlight php startinline %}  
class City extends \Illuminate\Database\Eloquent\Model {
    protected $table = 'city_copy';
    protected $fillable = array('name', 'full_name');
}

/*1*/
City::create(array('name' => 'bbb', 'xx' => 'xxxxx'));
// xx 会被过滤掉

/*2*/
$user = City::firstOrCreate(array('name' => 'John2'));    // 查找name=John2的用户，若没有则新增并取得新的实例...

/*3*/
$city = City::find(469); // or first()
$city->update(array('full_name' => 2)); // ps 如果要更新的字段跟原始的字段相同，则不会执行sql语句

// 动态添加白名单
$city->fillable(array('name', 'full_name', 'province_id'))->update(array('full_name' => 2));
{% endhighlight %}

#### 关于对象与json_encode()的关系
{% highlight php startinline %} 
JsonSerializable {
    /* 方法 */
    abstract public mixed jsonSerialize ( void )
}

//具体查看查看鸟哥的博客
//http://www.laruence.com/2011/10/10/2204.html

// Illuminate\Database\Eloquent\Model 部分方法
abstract class Model implements JsonSerializable, ArrayableInterface
{
    /**
     * 当使用json_encode($model); 的时候  $model会返回指定的数据作为json_encode的参数返回
     * @return array
     */
    public function jsonSerialize()
    {
        return $this->toArray();
    }

    /**
     * Convert the model instance to an array.
     * @return array
     */
    public function toArray()
    {
        $attributes = $this->attributesToArray();
        return array_merge($attributes, $this->relationsToArray());
    }
}

//Illuminate\Support\Collection Collection部分方法
class Collection implements JsonSerializable {

    /**
     * Get the collection of items as a plain array.
     *
     * @return array
     */
    public function toArray()
    {
        return array_map(function($value)
        {
            // model 也实现ArrayableInterface
            return $value instanceof ArrayableInterface ? $value->toArray() : $value;

        }, $this->items);
    }

    /**
     * 当使用json_encode($collectionObj); 的时候 $collectionObj会返回指定的数据作为json_encode的参数返回
     * @return array
     */
    public function jsonSerialize()
    {
        return $this->toArray();
    }
}
{% endhighlight %}

#### 其他
{% highlight php startinline %} 

 $carSeries = $model->where(function($query) use ($type) {
                    $query->whereIn('id', function($q) use ($type) { // 子查询
                        $q->select('series_id')->from('car_model')->where('type_id', $type);
                    })->orWhere('type_id', $type);

                });
{% endhighlight %}