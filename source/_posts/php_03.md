title: 按照权重随机
tags:
  - php
category: php
date: 2015-08-10 16:07:00

---
## code
按照权重随机,直接贴代码吧，思路大概就是在有限长线段上取一点，点落到某个区域内的概率有多大

```
<?php   
/**  
 * @param array $weight 权重	 例如array('a'=>10,'b'=>20,'c'=>50)  
 * @return string key   键名   
 */  
function roll($weight = array()) {   
	$roll = rand ( 1, array_sum ( $weight ) );   
	$_tmpW = 0;   
	$rollnum = 0;   
 	foreach ( $weight as $k => $v ) {   
		$min = $_tmpW;   
	 	$_tmpW += $v;   
	 	$max = $_tmpW;   
	 	if ($roll > $min && $roll <= $max) {   
	 		$rollnum = $k;   
	 		break;   
	 	}   
	}   
 	return $rollnum;   
}   
 
$row=roll(array('a'=>10,'b'=>20,'c'=>50));   
echo $row;   
?>
```