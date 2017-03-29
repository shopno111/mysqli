# mysqli
MySQL Funciton

<?php
//database common paramiter
//curret time
define("now","now()");
/**
 * @return int
 */

$con = (mysqli_connect( dbhost, dbuser, dbpass, dbname ));
function connectDb()
{
    GLOBAL $con;
    if($con==true)
    {
        return 1;
    }
    else
    {
        echo "Failed to connect to MySQL: " . mysqli_connect_error();
    }
}

function DBError()
{
	return mysqli_error();
}

function closeDB()
{
	mysqli_close();
}
function fixTables($dbname) 
{
    // search for all the tables of
    // a db and run repair and optimize
    // note: this can take a lot of time
    // if you have big/many tables.
    $result = mysql_list_tables($dbname) or die(mysqli_error());// prblm detected
    while ($row = mysqli_fetch_row($result)) {
        mysqli_query("REPAIR TABLE $row[0]");
        mysqli_query("OPTIMIZE TABLE $row[0]");
    }
}
//Table First column name
function tableID($table)
{
    GLOBAL $con;
	$query = mysqli_query($con,"SHOW COLUMNS FROM $table ");
	$qry = mysqli_fetch_array($query);
	return $qry[0];
}

function getHtmlTable($result){
    // receive a record set and print
    // it into an html table
    $out = '<table>';
    for($i = 0; $i < mysqli_num_fields($result); $i++){
        $aux = mysql_field_name($result, $i); ///
        $out .= "<th>".$aux."</th>";
    }
    while ($linea = mysqli_fetch_array($result, MYSQL_ASSOC)) {
        $out .= "<tr>";
        foreach ($linea as $valor_col) $out .= '<td>'.$valor_col.'</td>';
        $out .= "</tr>";
    }
    $out .= "</table>";
    return $out;
}

function getCommaFields( $table, $excepts = ""){
    // get a string with the names of the fields of the $table,
    // except the onews listed in '$excepts' param
    $out = "";
    $result = mysqli_query( "SHOW COLUMNS FROM `$table`" );
    while($row = mysqli_fetch_array($result)) if ( !stristr(",".$row['Field']."," , $excepts) ) $out.= ($out?",":"").$row['Field'];
    return $out ;
}

function getCommaValues($sql) {
    // execute a $sql query and return
    // all the first value of the rows in
    // a comma separated string
    $out = "";
    $rs = mysqli_query($sql) or die(mysqli_error().$sql);
    while($r=mysqli_fetch_row($rs)) $out.=($out?",":"").$r[0];
    return $out;
}

function getEnumSetValues( $table , $field ){
    // get an array of the allowed values
    // of the enum or set $field of $table
    $query = "SHOW COLUMNS FROM `$table` LIKE '$field'";
    $result = mysqli_query( $query ) or die( 'error getting enum field ' . mysqli_error() );
    $row = mysqli_fetch_array($result);
    if(stripos(".".$row[1],"enum(") > 0) $row[1]=str_replace("enum('","",$row[1]);
        else $row[1]=str_replace("set('","",$row[1]);
    $row[1]=str_replace("','","\n",$row[1]);
    $row[1]=str_replace("')","",$row[1]);
    $ar = split("\n",$row[1]);
    for ($i=0;$i<count($ar);$i++) $arOut[str_replace("''","'",$ar[$i])]=str_replace("''","'",$ar[$i]);
    return $arOut ;
}

function getScalar($sql,$def="") {
    // execute a $sql query and return the first
    // value, or, if none, the $def value
    $rs = mysqli_query($sql) or die(mysqli_error().$sql);
    if (mysqli_num_rows($rs)) {
        $r = mysqli_fetch_row($rs);
        mysqli_free_result($rs);
        return $r[0];
    }
    return $def;
}

function getRow($sql) {
    // execute a $sql query and return the first
    // row, or, if none, return an empty string
    $rs = mysqli_query($sql) or die(mysqli_error().$sql);
    if (mysqli_num_rows($rs)) {
        $r = mysqli_fetch_array($rs);
        mysqli_free_result($rs);
        return $r;
    }
    mysqli_free_result($rs);
    return "";
}

function duplicateRow($table,$primaryField,$primaryIDvalue) {
    // duplicate one record in a table
    // and return the id
    $fields = getCommaFields($table,$primaryField);
    $sql = "insert into $table ($fields) select $fields from $table where $primaryField='".mysql_real_escape_string($primaryIDvalue)."' limit 0,1";
    mysqli_query($sql) or die(mysqli_error().$sql);
    return mysqli_insert_id();
}

function convertResult($rs, $type, $jsonmain="") {
    // receive a recordset and convert it to csv
    // or to json based on "type" parameter.
    $jsonArray = array();
    $csvString = "";
    $csvcolumns = "";
    $count = 0;
    while($r = mysqli_fetch_row($rs)) {
        for($k = 0; $k < count($r); $k++) {
            $jsonArray[$count][mysql_field_name($rs, $k)] = $r[$k];
            $csvString.=",\"".$r[$k]."\"";
        }
        if (!$csvcolumns) for($k = 0; $k < count($r); $k++) $csvcolumns.=($csvcolumns?",":"").mysql_field_name($rs, $k);
        $csvString.="\n";
        $count++;
    }
    $jsondata = "{\"$jsonmain\":".json_encode($jsonArray)."}";
    $csvdata = str_replace("\n,","\n",$csvcolumns."\n".$csvString);
    return ($type=="csv"?$csvdata:$jsondata);
}

function QueryID($id,$table,$fld)
{
    GLOBAL $con;
	$tablID = tableID($table);
	$query = mysqli_query($con,"SELECT $fld from $table where $tablID='$id' ");
	$row = mysqli_fetch_assoc($query);
	return $row[$fld];
}

function QueryF($table,$cond,$fld)
{
    GLOBAL $con;
	$query = mysqli_query($con,"SELECT $fld from $table $cond ");
	$row = mysqli_fetch_assoc($query);
	return $row[$fld];
}

/**
 * @param $sql
 * @return bool|mysqli_result
 */
function Querys($sql)
{
    GLOBAL $con;
	$query = mysqli_query($con,$sql);
    return $query;
}
function QuerySS($table,$where,$fld)
{
    GLOBAL $con;
    $sql = "SELECT $fld from $table $where";
    $query = mysqli_query($con,$sql);
    if($query==true)
    {
        return $query;
    }
    else
    {
        return mysqli_error();
    }
}
function rowFetch($query)
{
	$assoc = mysqli_fetch_assoc($query);
	return $assoc;
}

//row count
function RowCount($fld,$table,$whr)
{
    GLOBAL $con;
	$query = mysqli_query($con,"SELECT $fld from $table $whr ");
	return mysqli_num_rows($query);
}

//summation
function QuerySum($fld,$table,$whr)
{
    GLOBAL $con;
	$query = mysqli_query($con,"SELECT $fld as sumvalue from $table $whr ");
	$row = mysqli_fetch_assoc($query);
	if($row['sumvalue']=='')
	{
		return '0.00';
	}
	else
	{
		return $row['sumvalue'];
	}
}
//conditional query
function QueryCond($table,$fld,$cond)
{
    GLOBAL $con;
	$query = mysqli_query($con,"SELECT $fld from $table $cond ");
	$row = mysqli_fetch_assoc($query);
	return $row[$fld];
}


//query filtering for single row
function QueryIDS($id,$table,$fld)
{
    GLOBAL $con;
	$query = mysqli_query($con,"SELECT $fld from $table where id='$id' ");
	$row = mysqli_fetch_assoc($query);
	return $row[$fld];
}

//getall fld query
function getAllFld($table,$fld,$whr)
{
    GLOBAL $con;
	$query = mysqli_query($con,"SELECT * from $table where $whr ");
	$row = mysqli_fetch_assoc($query);
	return $row;
}

//mysql update

function DBupdate($table,$fld1,$fld2,$whr)
{
    GLOBAL $con;
	if($fld2 !='')
	{
		$qry = $fld1.', '.$fld2;
	}
	else
	{
		$qry = $fld1;
	}
	$update = mysqli_query($con,"UPDATE $table SET $qry WHERE $whr ");
	$qr = ("Update $table SET $qry where $whr ");
	$error = mysqli_error();
	if($update==true)
	{
		return '1';
	}
	else
	{
		return '2'.$error.'|'.$qr;
	}
}



//mysql INSERT

function DBinsert($table,$fld)
{
    GLOBAL $con;
	$update = mysqli_query($con,"INSERT INTO $table $fld ");
	$insid = mysqli_insert_id($con);
	if($update==true)
	{
		$return = '1:'.$insid;
		return $return;
	}
	else
	{
		$return = '2:'.mysqli_error();
		return $return;
	}
}

//db DELETE

function DBdelete($table,$whr)
{
    GLOBAL $con;
	$query = mysqli_query($con,"Delete from $table where $whr ");
	
	if($query==true)
	{
		$return = '1';
		return $return;
	}
	else
	{
		$return = '2:'.mysqli_error();
		return $return;
	}
}

//row check
function RowCheck($sql)
{
    GLOBAL $con;
	$query = mysqli_query($con,$sql);
	return mysqli_num_rows($query);
}

//duplicate check
function dupCheck($table,$fld,$value)
{
    GLOBAL $con;
	$query = mysqli_query($con,"SELECT $fld from $table where $fld='$value' ");
	if(mysql_num_rows($query)>0)
	{
		return 1;
	}
	else
	{
		return 0;
	}
}
?>
