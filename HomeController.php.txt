<?php

namespace App\Http\Controllers\Site;

use App\Models\Complaint;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Redirect;
use Illuminate\Support\Facades\DB;
use Validator;
use Illuminate\Support\Facades\Input;

class HomeController extends Controller
{
    public function getHome(){
        return view('site.app');
    }


    public function Add(Request $request){

        if($request->full_name == null and $request->email == null){
            return Redirect::back()->with('msg', '<span style="color: red;">Xaiş edirik adınızı və email yazın</span>');
        }

//        Before insert data in DB chek format docs
        if($request->file() != null) {
            $files = $request->file();
            foreach ($files['file'] as $file) {
                $fileArray = array('file' => $file);
                $rules = array(
                    'file' => 'mimes:pdf,doc,dot,wbk,docx,docm,dotx,dotm,docb,xls,xlt,xlm,xlsx,xlsm,xltx,xltm,xlsb,xla,xlam,xll,xlw,ppt,pot,pps,pptx,pptm,potx,potm,ppam,ppsx,ppsm,sldx,sldm,jpeg,jpg,png,gif|required|max:10000' // max 10000kb
                );
                $validator = Validator::make($fileArray, $rules);
                if ($validator->fails()) {
                    $messages = $validator->errors()->getMessages();
                    return Redirect::back()->with('error', $messages['file'][0]);
                }
            }
        }
//        end



        if($request->yes){
            $yes = $request->yes;
        }else{
            $yes = 0;
        }
        $complaint = new Complaint();
        $complaint->full_name           = strip_tags($request->full_name);
        $complaint->adress              = strip_tags($request->adress);
        $complaint->city                = strip_tags($request->adress);
        $complaint->country             = strip_tags($request->country);
        $complaint->telephone           = strip_tags($request->telephone);
        $complaint->email               = $request->email;
        $complaint->education           = strip_tags($request->education);
        $complaint->date_problem        = strip_tags($request->date_problem);
        $complaint->complaint_subject   = strip_tags($request->complaint_subject);
        $complaint->message             = strip_tags($request->message);
        $complaint->yes                 = $yes;
        $complaint->date                = date('Y-m-d');
        $complaint->time                = date("H:i:s");
        if($complaint->save()){
            //email send kod will be here

            $complaint_kod = DB::table('complaint')->orderBy('id', 'desc')->first();
            $complaint_kod = $complaint_kod->id;

//            upload files
            if($request->file() != null) { //chek file is not null
                $files = $request->file();
                $imgs = array();
                $i = 0;
                foreach ($files['file'] as $file) {
                    // Build the input for validation
                    $fileArray = array('file' => $file);
                    // Tell the validator that this file should be an image
                    $rules = array(
                        'file' => 'mimes:txt,pdf,doc,dot,wbk,docx,docm,dotx,dotm,docb,xls,xlt,xlm,xlsx,xlsm,xltx,xltm,xlsb,xla,xlam,xll,xlw,ppt,pot,pps,pptx,pptm,potx,potm,ppam,ppsx,ppsm,sldx,sldm,jpeg,jpg,png,gif|required|max:10000' // max 10000kb
                    );
                    // Now pass the input and rules into the validator
                    $validator = Validator::make($fileArray, $rules);
                    // Check to see if validation fails or passes
                    if ($validator->fails()) {
                        // Redirect or return json to frontend with a helpful message to inform the user
                        // that the provided file was not an adequate type
//            return response()->json(['error' => $validator->errors()->getMessages()], 400);
                        $messages = $validator->errors()->getMessages();
                        return Redirect::back()->with('error', $messages['file'][0]);
                    } else {
                        if ($file->isValid()) {
                            $destinationPath = public_path() . '/docs/'; // upload path
                            $extension = $file->getClientOriginalExtension(); // getting image extension
                            $fileName = $complaint_kod.'_'.uniqid() . '.' . $extension; // renameing image
                            $file->move($destinationPath, $fileName); // uploading file to given path
                            $imgs[$i] = $fileName;
                        }
                    };
                    $i++;
                }
//            end uplaod files kod

                foreach ($imgs as $img) {
                    DB::table('docs')->insert([
                        'name' => $img,
                        'complaint_id' => $complaint_kod,
                    ]);
                }
            }


            return Redirect::back()->with('msg', '<span style="color: #4cae4c;">Success! Your Kod</span> <strong style="color: red;">'.$complaint_kod.'</strong>');
        }else{
            return Redirect::back()->with('msg', '<span style="color: #880000;">Not saved!</span>');
        }
    }



    public function Satisfaction(){
        return view('site.satisfaction');
    }


}
