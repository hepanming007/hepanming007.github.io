---
layout: post
title: 无废话Yii2自定义异常处理
category: php
tags: php,yii2,异常处理
description: yii2异常处理
---
 

###1.创建自定义errorHandler
<pre class="prettyprint">
namespace app\components\exception;
use Yii;
class ErrorHandler extends \yii\base\ErrorHandler{

    public $errorView = '@app/views/errorHandler/error.php';
    public function renderException($exception)
    {
        if(Yii::$app->request->getIsAjax()){
            exit( json_encode( array('code' =>$exception->getCode(),'msg'  =>$exception->getMessage()) ));
        }else{
            echo  Yii::$app->getView()->renderFile($this->errorView,['exception' => $exception,],$this);
        }
    }
}
</pre>


###2.web.php 配置里头添加错误处理errorHandler
<pre class="prettyprint">
 $config=[
   'components'=>
     'errorHandler' => [
        'class'=>'app\components\exception\ErrorHandler'
 ]
</pre>


视图文件

<pre class="prettyprint">
$message = [
    'code'=>$exception->getCode(),
    'msg'=>$exception->getMessage(),
];
print_r($message);
</pre>

###3.自定义异常

<pre class="prettyprint">
namespace app\components\exception;
use yii\base\UserException;

class FileException extends UserException{
    CONST UPLOAD_SUECESS= 1000;
    CONST FILE_SIZE_TO_LARGE = 2000;
    static $codeMessage = array(
        '1000'=>'上次成功',
        '2000'=>'文件超出大小',
    );

    public function __construct($code = 0, \Exception $previous = null)
    {
        parent::__construct(self::$codeMessage[$code], $code, $previous);
    }

    public function getName()
    {
        return 'FileException';
    }
}
</pre>
###4.抛异常

<pre class="prettyprint">
namespace app\controllers;
use yii;
use yii\web\Controller;
use app\components\exception\FileException;
class ExceptionController extends Controller{

        public function actionTest()
        {
             throw new FileException(FileException::FILE_SIZE_TO_LARGE);
        }
}
</pre>
