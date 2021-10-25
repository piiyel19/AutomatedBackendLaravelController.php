# AutomatedBackendLaravelController.php
Aku malas nak menulis kod yang berulang ! Nah follow jer teknik ni !


-> klik sini bro : https://github.com/piiyel19/AutomatedBackendLaravelController.php/blob/main/README.md?plain=1

\n



<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Redirect;

use File;

class AutoBackendController extends Controller
{
	public function __construct()
    {
        $this->middleware('auth');
    }
    
    //
    public function index()
    {
    	\Artisan::call('optimize');
    	dd('optimize successfully');

        //return redirect()->route('form_automated');

    }


    public function make_controller($name)
    {	
    	if(!empty($name)){
	    	\Artisan::call('make:controller '.$name.'/'.$name.'Controller');
	    	//dd('controller created successfully');

            
	    }
    }


    public function make_model($name)
    {	
    	if(!empty($name)){
	    	\Artisan::call('make:model Models/'.$name.'/'.$name.'Model');
	    	//dd('model created successfully');
	    }
    }


    public function make_migration($table)
    {	
    	//dd($table);
    	if(!empty($table)){
    		$table = strtolower($table);
	    	\Artisan::call('make:migration create_'.$table.'_table');
	    	//dd('migration created successfully');
	    }
    }


    public function write_controller($class,$parameter)
    {
    	$path_base = 'C:\inetpub\wwwroot\{namaProject}\app\Http\Controllers';
    	// $file = '\TestController.php';

    	$slash = "/";
    	$slash = str_replace('/', '\\', $slash);

    	$filename = $slash.$class;
    	$file = 'Controller.php';

    	$path = $path_base.$filename.$filename.$file;

    	//dd($parameter);

    	$myfile = fopen($path, "w") or die("Unable to open file!");


    	// $parameter = array('$name'=>'name','$age'=>'age');
    	$where = array('$id'=>'id');

    	// get Variable 
    	$declare_variable = '';
    	$create_query = '';
    	foreach ($parameter as $key_params => $params) {
    		# code...

    		$req = "$ request";
    		$declare_variable .="$key_params=".$req."->".$params.';';

    		$create_query .= "'".$params."' => $key_params,";

    	}
    	$declare_variable = str_replace(' ', '', $declare_variable);
    	$create_query = str_replace(' ', '', $create_query);
    	$create_query = rtrim($create_query, ',');
    	//dd($declare_variable);


    	// get ID
    	$declare_where = '';
    	$where_query = '';
    	$delete_query = '';
    	$i=0;
    	foreach ($where as $key_where => $where) {
    		# code...

    		$req = "$ request";
    		$declare_where .="$key_where=".$req."->".$where.';';

    		if($i==0){
    			$where_query .="where('".$where."',$key_where)";
    		} else {
    			$where_query .="->where('".$where."',$key_where)";
    		}

    		$delete_query .="->where('".$where."',$key_where)";
    	
    		$i++;
    	}
    	$declare_where = str_replace(' ', '', $declare_where);
    	$where_query = str_replace(' ', '', $where_query);
    	$delete_query = str_replace(' ', '', $delete_query);
    	//dd($declare_variable);


    	$request_declare = 'Request $request';
    	$id_params = '$id';


    	$fileModel = $slash.$class.'Model';

    	$txt = '<?php
    			// Library Declare
				namespace App\Http\Controllers;
				use Illuminate\Http\Request;
				use Illuminate\Support\Facades\Auth;
				use Illuminate\Support\Facades\Redirect;
				use Session;
				// Model Name
				use App\Models'.$filename.$fileModel.';
				class '.$class.'Controller extends Controller
				{
					public function __construct()
				    {
				        $this->middleware("auth");
				        date_default_timezone_set("Asia/Kuala_Lumpur"); //Timezone
        				//$date = date("Y-m-d H:i:s");
				    }
				    //
					public function index()
					{
						return view("'.$class.'/index");
					}
					public function ui_update()
					{
						return view("'.$class.'/update");
					}
					public function ui_list()
					{
						return view("'.$class.'/list");
					}
				    public function create('.$request_declare.')
				    {
				    	'.$declare_variable.'
				    	User::create([
				            '.$create_query.'
				        ]);
				    }
				    public function update('.$request_declare.')
				    {
				    	'.$declare_variable.'
				    	'.$declare_where.'
				    	User::'.$where_query.'
				    	->update([
				    		'.$create_query.'
				    	]);
				    }
				    public function delete('.$request_declare.')
				    {
				    	'.$declare_where.'
				    	DB::table("User")'.$delete_query.'->delete();
				    }
				}
    	';

    	fwrite($myfile, "\n". $txt);
    	fclose($myfile);

    }



    public function write_model($class,$parameter)
    {
    	$path_base = 'C:\inetpub\wwwroot\{namaProject}\app\Models';
    	//$file = '\TestModel.php';

    	$slash = "/";
    	$slash = str_replace('/', '\\', $slash);

    	$filename = $slash.$class;
    	$file = 'Model.php';

    	$path = $path_base.$filename.$filename.$file;

    	//$path = $path_base.$file;

    	$myfile = fopen($path, "w") or die("Unable to open file!");


    	//$parameter = array('$id'=>'id','$name'=>'name','$age'=>'age');
    	// get Variable 
    	$declare_name = '';
    	foreach ($parameter as $key_params => $params) {
    		# code...
    		$declare_name .= '"'.$params.'",';
    	}
    	$declare_name = str_replace(' ', '', $declare_name);
    	$declare_name = rtrim($declare_name, ',');

    	$txt = 
    	'
    		<?php
			namespace App\Models'.$filename.';
			use Illuminate\Database\Eloquent\Factories\HasFactory;
			use Illuminate\Database\Eloquent\Model;
			use Illuminate\Notifications\Notifiable;
			class '.$class.'Model extends Model
			{
			    use HasFactory;
			    use Notifiable;
			    protected $fillable = ['.$declare_name.'];
			}
    	';

    	fwrite($myfile, "\n". $txt);
    	fclose($myfile);
    }


    public function form()
    {
    	return view('form_automated');
    }

    public function automate(Request $request)
    {
    	$table = $request->table;
    	$param = $request->param;
    	$name = $request->name;
    	$type = $request->type;

    	$len = count($param);

    	$parameter = array();

    	$table = str_replace(' ', '', $table);


    	for($i=0;$i<$len;$i++)
    	{
    		$param_data = $param[$i];
    		$name_data = $name[$i];
    		$type_data = $type[$i];

    		$param_data = str_replace(' ', '', $param_data);
    		$name_data = str_replace(' ', '', $name_data);
    		$type_data = str_replace(' ', '', $type_data);
    		
    		$parameter["$"."$param_data"] = $name_data;

    	}


    	// create controller
    	$class = ucfirst($table);


    	$slash = "/";
    	$slash = str_replace('/', '\\', $slash);

    	$filename = $slash.$class;

    	

    	$path_folder = 'C:/inetpub/wwwroot/{namaProject}/resources/views'.$filename;

    	//dd($path_folder);
    	if (!file_exists($path_folder)) {
		    File::makeDirectory($path_folder, 755, true);
		}


        // controller
        $path_folder = 'C:/inetpub\wwwroot/{namaProject}/app/Http/Controllers'.$filename;

        //dd($path_folder);
        if (!file_exists($path_folder)) {
            File::makeDirectory($path_folder, 755, true);
        }



        // model
        $path_folder = 'C:/inetpub\wwwroot/{namaProject}/app/Models'.$filename;

        //dd($path_folder);
        if (!file_exists($path_folder)) {
            File::makeDirectory($path_folder, 755, true);
        }




		$path_file = 'C:/inetpub/wwwroot/{namaProject}/resources/views'.$filename.'/index.blade.php';

		$newFileContent = 'Test';
		if (file_put_contents($path_file, $newFileContent) !== false) {
		    echo "File created (" . basename($path_file) . ")";
		} else {
		    echo "Cannot create file (" . basename($path_file) . ")";
		}


		$path_file = 'C:/inetpub/wwwroot/{namaProject}/resources/views'.$filename.'/list.blade.php';

		$newFileContent = 'Test';
		if (file_put_contents($path_file, $newFileContent) !== false) {
		    echo "File created (" . basename($path_file) . ")";
		} else {
		    echo "Cannot create file (" . basename($path_file) . ")";
		}



		$path_file = 'C:/inetpub/wwwroot/{namaProject}/resources/views'.$filename.'/update.blade.php';

		$newFileContent = 'Test';
		if (file_put_contents($path_file, $newFileContent) !== false) {
		    echo "File created (" . basename($path_file) . ")";
		} else {
		    echo "Cannot create file (" . basename($path_file) . ")";
		}




    	$this->make_controller($class);
    	$this->make_model($class);
    	//$this->make_migration($class);
    	$this->write_controller($class,$parameter);
    	$this->write_model($class,$parameter);


        dd("Success created backend Controller , Model & View !");

    }
}
