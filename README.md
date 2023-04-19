# RegistrationForm-Using-Room-data
package com.example.registrationandloginpage
import android.app.ProgressDialog
import android.content.Intent
import android.os.Bundle
import android.os.Handler
import androidx.appcompat.app.AppCompatActivity
import androidx.room.Room
import com.example.registrationandloginpage.databinding.ActivityMainBinding



class MainActivity : AppCompatActivity() {
    lateinit var binder: ActivityMainBinding
    private lateinit var db:AppDatabase    // declaring app database
    private lateinit var modelDaoClass: ModelDaoClass   // declaring modelDaoClass

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binder= ActivityMainBinding.inflate(layoutInflater)
        setContentView(binder.root)
  // Use of Progress Bar
        val prg=ProgressDialog(this)
        prg.setCancelMessage("Please Wait While We Connect")
        prg.setCancelable(false)
       Handler().postDelayed({prg.dismiss()},3000)
        prg.show()

      // use of Marquee


        binder.btnlogin.setOnClickListener { //      Login button
            if (validation()){
                login()
            }


        }
        binder.btnreg.setOnClickListener {   //      Register Button
            register()
        }

    }

    fun login(){

        var userName = binder.userEmail.text.toString()
        var userPassword = binder.password.text.toString()


            lateinit var dataClass: DataClass

        var odb = Room.databaseBuilder(   // creating  room database
            applicationContext,
            AppDatabase::class.java, "database-name"
        ).allowMainThreadQueries()      //     allow the main thread
            .build()
        val db = AppDatabase.getInstance(this) // calling the instance method
        modelDaoClass = db.modelDaoClass()

        // Call the DAO method to retrieve all data
            val allUsers = db.modelDaoClass().getall()  // calling the method of dao class

        for (user in allUsers) {
            if (user.userPassword == userPassword) {
                val intent = Intent(this, LoginButton::class.java)
                val bundle = Bundle()
                bundle.putString("username", user.userName)
                bundle.putString("useremail", user.userEmail)
                bundle.putString("userpass", user.userPassword)
                bundle.putString("userphone", user.userPhone.toString())
                bundle.putString("userdob", user.userDob)
                bundle.putString("usergen", user.UserMale)
                intent.putExtras(bundle)
                startActivity(intent)
                break
            }
        }

        // Do something with the list of users


    }

    fun register(){
        val intent = Intent(this,Registerbutton::class.java)
        startActivity(intent)
    }

    fun validation():Boolean {
        if (binder.userEmail.length() == 0) {
            binder.userEmail.error = "please fill the name "
            return false
        }
        if (binder.password.length()==0) {
            binder.password.error = "please fill the Password "
            return false
        }
        return true
    }
}

private fun ProgressDialog.setCancelMessage(s: String) {

}

--------------------------------------------------------------------------------------
package com.example.registrationandloginpage

import android.content.Context
import androidx.lifecycle.ViewModelProvider.NewInstanceFactory.Companion.instance
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.sqlite.db.SupportSQLiteDatabase

@Database(entities = [DataClass::class], version = 2)
abstract class AppDatabase:RoomDatabase() {  // creation of a room database     ---1st step
// Here we are extending the room database i.e. RoomDatabase()

    abstract fun modelDaoClass(): ModelDaoClass          // ---       2nd step

    companion object {
        private var instance: AppDatabase? = null    // creating instance object  -- 3rd step

        fun getInstance(context: Context): AppDatabase {
            // get instance by creating method getInstance  ---4th step

            if (instance == null) {   // if it is null then we create a room database --5th step
                instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                ).allowMainThreadQueries().fallbackToDestructiveMigration().build()
            }
            return instance as AppDatabase  // return instance --6th step
        }

    }

}------------------------------------------------------------------------------------------

package com.example.registrationandloginpage

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "reg_table")

data class DataClass(
    @PrimaryKey
    @ColumnInfo(name = "phone") val userPhone: String,
    @ColumnInfo(name = "Name") val userName: String,
    @ColumnInfo(name = "Email") val userEmail: String,
    @ColumnInfo(name = "Password") val userPassword:String,
    @ColumnInfo(name = "Dob") val userDob: String,
    @ColumnInfo(name = "Male") val UserMale: String,
    @ColumnInfo(name = "Female") val userFemale: String



    )
----------------------------------------------------------------------------------------------------

package com.example.registrationandloginpage

import android.content.Intent
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import com.example.registrationandloginpage.databinding.ActivityLoginButtonBinding

class LoginButton : AppCompatActivity() {

    private lateinit var binder:ActivityLoginButtonBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binder= ActivityLoginButtonBinding.inflate(layoutInflater)
       val username= intent.getStringExtra("username")
        val useremail= intent.getStringExtra("useremail")
        val userpassword= intent.getStringExtra("userpass")
        val userdob= intent.getStringExtra("userdob")
        val usergen= intent.getStringExtra("usergen")
        val userphone= intent.getStringExtra("userphone")

        binder.username.setText(username)
        binder.userEmail.setText(useremail)
        binder.userpassword.setText(userpassword)
        binder.userDob.setText(userdob)
        binder.userMale.setText(usergen)
        binder.userMobile.setText(userphone)
        setContentView(binder.root)

        binder.delete.setOnClickListener {

            clearData()

        }

    }
    private fun clearData(){
       /* binder.username.text.clear()
       binder.userEmail.text=("")
       binder.userpassword.text=("")
       binder.userDob.text=("")
       binder.userMobile.text.clear()
       binder.userMale.text=("")*/
        val intent = Intent(this,MainActivity::class.java)
        startActivity(intent)
    }
}

--------------------------------------------------------------
package com.example.registrationandloginpage
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Update

@Dao
interface ModelDaoClass {


    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insert(dataClass: DataClass)

    @Delete
    fun deleteAll(dataClass: DataClass)

    @Update
    fun update(dataClass: DataClass)

     @Query("select * from reg_table ")
     fun getall():List<DataClass>    // declaring the data class  name in dao class






}
-----------------------------------------------------------------------------

package com.example.registrationandloginpage

import android.annotation.SuppressLint
import android.app.AlarmManager
import android.app.AlertDialog
import android.app.DatePickerDialog
import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Toast
import com.example.registrationandloginpage.databinding.ActivityRegisterbuttonBinding
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.DelicateCoroutinesApi
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.launch
import java.util.*

class Registerbutton : AppCompatActivity() {
    lateinit var binder: ActivityRegisterbuttonBinding
    private lateinit var modelDaoClass: ModelDaoClass  // Declarinfg dao class
    private lateinit var appdb:AppDatabase   // declaring room database
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binder = ActivityRegisterbuttonBinding.inflate(layoutInflater)
        setContentView(binder.root)


        binder.button.setOnClickListener {  // Declaring Submit button of registration
            if (validation()){
                submit()
                alert()
            }
        }

        binder.userDob.setOnClickListener {
            age()
            validation()
        }
      /*  binder.userImage.setOnClickListener {  }*/
    }


    @OptIn(DelicateCoroutinesApi::class)
    @SuppressLint("SetTextI18n")
    fun submit(){      // binding all the Edit text one by one
        val userName = binder.username.text.toString()
        val userEmail = binder.userEmail.text.toString()
        val userPhone = binder.userMobile.text.toString()
        val userPassword = binder.userpassword.text.toString()
        val userDob = binder.userDob.text.toString()
        binder.userFeMale.text = "Female"
        binder.userMale.text = "Male"
        val female = binder.userFeMale.text
        val male = binder.userMale.text
        /*var image = binder.userImage.text.toString()*/



        val db = AppDatabase.getInstance(this)
        modelDaoClass = db.modelDaoClass()


        val userDao = db.modelDaoClass()
        CoroutineScope(GlobalScope.coroutineContext).launch {
            userDao.insert(DataClass(
                userName = userName, userEmail = userEmail, userPassword = userPassword
                , userPhone =userPhone, userDob =userDob, UserMale = male as String, userFemale = female as String
            ))
        }
        val intent = Intent(this,RegistrationSuccesfull::class.java)
        startActivity(intent)
        Toast.makeText(this, "Submitted", Toast.LENGTH_LONG).show()


    }


    fun age() {

        val myCalender = Calendar.getInstance() // by get Instance we are creating object
        val year = myCalender.get(Calendar.YEAR)
        val month = myCalender.get(Calendar.MONTH)
        val day = myCalender.get(Calendar.DAY_OF_MONTH)

        Toast.makeText(this, "Button clicked", Toast.LENGTH_LONG).show()

        DatePickerDialog(
            this,
            DatePickerDialog.OnDateSetListener { View, year, month, day -> },
            year,
            month,
            day).show()

        // calender Dialog box is used by DatePickerDialog
        val selectDate = ("$year/${month + 1}/$day")
        binder.userDob.setText(selectDate)

    }
    fun validation():Boolean {
        if (binder.username.length()==0) {
            binder.username.error = "please fill the name "
            return false
        }
        if (binder.userEmail.length()==0) {
            binder.userEmail.error = "please fill the Email "
            return false
        }
        if (binder.userpassword.length() ==0) {
            binder.userpassword.error = "please fill the Password "
            return false
        }
        if (binder.userMobile.length() ==0) {
            binder.userMobile.error = "please fill the Mobile "
            return false
        }
        if (binder.userDob.length() == 0) {
            binder.userDob.error = "please fill the DOB "
            return false
        }
        return true
    }

    fun alert(){
        val builder = AlertDialog.Builder(this)
        builder.setTitle("Alert")
        builder.setMessage("This is an alert message.")
        builder.setPositiveButton("OK") { dialog, which ->
            // do something when the "OK" button is clicked
        }

       builder.show()
    }
}
-----------------------------------------------------------------------------------
package com.example.registrationandloginpage

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import com.example.registrationandloginpage.databinding.ActivityRegisterbuttonBinding
import com.example.registrationandloginpage.databinding.ActivityRegistrationSuccesfullBinding

class RegistrationSuccesfull : AppCompatActivity() {
    lateinit var binder:ActivityRegistrationSuccesfullBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binder= ActivityRegistrationSuccesfullBinding.inflate(layoutInflater)
        setContentView(binder.root)

        binder.button.setOnClickListener{
            val intent = Intent(this,MainActivity::class.java)
            startActivity(intent)
        }

    }

}

---------------------------------------------------------------- XML File

<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/img_1"
    tools:context=".LoginButton">



    <TextView
        android:id="@+id/text1"
        android:layout_width="337dp"
        android:layout_height="63dp"
        android:layout_margin="30dp"
        android:layout_marginStart="20dp"
        android:layout_marginTop="16dp"
        android:padding="20dp"
        android:text="Login Details"
        android:textAlignment="center"
        android:textColor="@color/black"
        android:textSize="20sp"
        android:textStyle="bold"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card1"
        android:layout_width="300dp"
        android:layout_height="60dp"
        android:layout_margin="20dp"
        android:layout_marginStart="16dp"
        app:cardBackgroundColor="#2196F3"
        app:cardCornerRadius="36dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/text1" />

    <EditText
        android:id="@+id/username"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:layout_marginStart="16dp"
        android:drawableLeft="@drawable/baseline_supervised_user_circle_24"
        android:drawablePadding="10dp"
        android:elevation="10dp"
        android:hint="User Name"
        android:inputType="textEmailAddress"
        android:paddingLeft="20dp"
        android:textColor="@color/black"
        android:textSize="20sp"
        android:textStyle="bold"
        app:layout_constraintStart_toStartOf="@+id/card1"
        app:layout_constraintTop_toTopOf="@+id/card1" />

    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card3"
        android:layout_width="300dp"
        android:layout_height="60dp"
        android:layout_margin="20dp"
        app:cardBackgroundColor="#2196F3"
        app:cardCornerRadius="36dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card2" />

    <EditText
        android:id="@+id/userMobile"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:drawablePadding="10dp"
        android:layout_marginStart="188dp"
        android:layout_marginTop="236dp"
        android:drawableLeft="@drawable/baseline_phone_24"
        android:elevation="10dp"
        android:hint="User Phone"
        android:paddingLeft="20dp"
        android:textColor="@color/black"
        android:textSize="20sp"
        app:layout_constraintStart_toStartOf="@+id/card2"
        app:layout_constraintTop_toTopOf="@+id/card3" />


    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card4"
        android:layout_width="300dp"
        android:layout_height="60dp"
        android:layout_margin="20dp"
        app:cardBackgroundColor="#2196F3"
        app:cardCornerRadius="36dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card3" />

    <TextView
        android:id="@+id/userEmail"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:layout_marginStart="188dp"
        android:layout_marginTop="236dp"
        android:drawableLeft="@drawable/baseline_email_24"
        android:elevation="10dp"
        android:hint="User Email"
        android:drawablePadding="10dp"
        android:paddingLeft="20dp"
        android:textColor="@color/black"
        android:textSize="20sp"
        app:layout_constraintStart_toStartOf="@+id/card1"
        app:layout_constraintTop_toTopOf="@+id/card2" />

    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card2"
        android:layout_width="300dp"
        android:layout_height="60dp"
        android:layout_margin="20dp"
        app:cardBackgroundColor="#2196F3"
        app:cardCornerRadius="36dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card1" />

    <TextView
        android:id="@+id/userpassword"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:layout_marginStart="188dp"
        android:layout_marginTop="236dp"
        android:drawableLeft="@drawable/baseline_lock_24"
        android:drawablePadding="10dp"
        android:elevation="10dp"
        android:hint="User Password"
        android:paddingLeft="20dp"
        android:textColor="@color/black"
        android:textSize="20sp"
        app:layout_constraintStart_toStartOf="@+id/card3"
        app:layout_constraintTop_toTopOf="@+id/card4" />


    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card5"
        android:layout_width="300dp"
        android:layout_height="60dp"
        android:layout_margin="20dp"
        app:cardBackgroundColor="#2196F3"
        app:cardCornerRadius="36dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card4" />
    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/userDob"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="20dp"
        android:layout_marginStart="188dp"
        android:layout_marginTop="236dp"
        android:drawableLeft="@drawable/baseline_calendar_month_24"
        android:drawablePadding="10dp"
        android:elevation="10dp"
        android:hint="User Dob"
        android:paddingLeft="20dp"
        android:textColor="@color/black"
        android:textSize="20sp"
        android:focusable="false"
        app:layout_constraintStart_toStartOf="@+id/card4"
        app:layout_constraintTop_toTopOf="@+id/card5" />



    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card6"
        android:layout_width="300dp"
        android:layout_height="60dp"
        android:layout_margin="20dp"
        app:cardBackgroundColor="#2196F3"
        app:cardCornerRadius="36dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card5" />

    <TextView
        android:id="@+id/userMale"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="20dp"

        android:layout_marginStart="188dp"
        android:layout_marginTop="236dp"
        android:drawablePadding="10dp"
        android:elevation="10dp"
        android:hint="Gender"
        android:drawableLeft="@drawable/baseline_transgender_24"
        android:paddingLeft="20dp"
        android:textColor="@color/black"
        android:textStyle="bold"
        android:textSize="20sp"
        android:focusable="false"
        app:layout_constraintStart_toStartOf="@+id/card5"
        app:layout_constraintTop_toTopOf="@+id/card6" />

    <Button
        android:id="@+id/delete"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="24dp"
        android:backgroundTint="#2196F3"
        android:gravity="center"
        android:text="Log-Out"
        android:textAlignment="center"
        android:textColor="@color/black"
        android:textStyle="bold"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.498"
        app:layout_constraintStart_toStartOf="parent" />


</androidx.constraintlayout.widget.ConstraintLayout>

--------------------------------------------------------------------------
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/img_3"
    tools:context=".RegistrationSuccesfull">


    <Button

        android:id="@+id/button"
        android:layout_width="300dp"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:backgroundTint="@color/white"
        android:gravity="center"
        android:text="Go To Login Page"
        android:textColor="@color/yellow"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.407"
        app:layout_constraintStart_toStartOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>

-------------------------------------------------------------------------
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:background="@drawable/img"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    

    <TextView
        android:id="@+id/text1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="30dp"
        android:padding="20dp"
        android:text="@string/login"
        android:textSize="20sp"
        android:textColor="@color/black"
        android:textStyle="bold"
        android:textAlignment="center"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card1"
        android:layout_width="310dp"
        android:layout_height="70dp"
        android:layout_margin="40dp"
        app:cardBackgroundColor="#2196F3"
        app:cardCornerRadius="36dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/text1" />

    <EditText
        android:id="@+id/userEmail"
        android:elevation="10dp"
        android:layout_width="280dp"
        android:layout_height="wrap_content"
        android:paddingLeft="20dp"
        android:drawablePadding="10dp"
        android:layout_margin="16dp"
        android:drawableLeft="@drawable/baseline_email_24"
        android:hint="User Email"
        android:inputType="textEmailAddress"
        android:textSize="20sp"
        android:textStyle="bold"
        android:textColor="@color/black"
        app:layout_constraintStart_toStartOf="@+id/card1"
        app:layout_constraintTop_toTopOf="@+id/card1"
        />

    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card2"
        android:layout_width="310dp"
        android:layout_height="70dp"
        android:layout_margin="40dp"
        app:cardBackgroundColor="#2196F3"
        app:cardCornerRadius="36dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card1" />

    <EditText
        android:id="@+id/password"
        android:layout_width="280dp"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:drawablePadding="10dp"
        android:textStyle="bold"
        android:layout_marginStart="188dp"
        android:layout_marginTop="236dp"
        android:drawableLeft="@drawable/baseline_lock_24"
        android:elevation="10dp"
        android:hint="User Password"
        android:paddingLeft="20dp"
        android:textColor="@color/black"
        android:textSize="20sp"
        app:layout_constraintStart_toStartOf="@+id/card1"
        app:layout_constraintTop_toTopOf="@+id/card2" />

    <Button
        android:id="@+id/btnlogin"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="30dp"
        android:drawableLeft="@drawable/baseline_login_24"
        android:layout_marginLeft="100dp"
        android:textColor="@color/black"
        android:layout_marginTop="140dp"
        android:text="  Login  "
        android:backgroundTint="#002E51"
        android:textSize="20sp"
        android:textAllCaps="false"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card2" />

    <Button
        android:id="@+id/btnreg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="5dp"
        android:text=" Register"
        android:drawableLeft="@drawable/baseline_app_registration_24"
        android:textColor="@color/black"
        android:backgroundTint="#53085A"
        android:textSize="20sp"
        android:textAllCaps="false"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card2" />



</androidx.constraintlayout.widget.ConstraintLayout>

---------------------------------------------------------------------------------
