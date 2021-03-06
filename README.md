/*Group Name: C27.  Group Members: Joseph Cashmore, Joe Colebrook, Alex Naylor.*/
/*This program works best if the command window width is set to size 200 by default.*/

#include <stdio.h>          /* The libraries needed for the code below.*/
#include <string.h>
#include <grx20.h>
#include <math.h>
#include <windows.h>

const int pressure_at_sea_level = 101325;
const int radius_of_earth = 6371000;
const float gravitational_constant = 6.67384E-11;       /* Declaration of global constants, with E being used instead of "pow", and doubles*/
const float mass_of_earth = 5.972E24;                   /* being used for increased precision.*/
const float gas_constant = 8.31;
const double PI = 3.141592654;
float time;                               /* Declaration of global variables, this is so that they are updated within the data structure*/
float density;                            /* when calculations are carried out.*/  
float mass;
float gravity;
float altitude;
float drag;
float acceleration;
float velocity;
 
const int MAXROW = 250, MAXCOL = 8, COLt = 1, COLRo = 2, COLdrag = 3, COLa = 4, COLv = 5, COLh = 6, COLg = 7, COLm = 8;	


typedef struct	RSIMStructure {     /* Defining the data structure that is used throughout the program.*/               
    int	currentrow;                 /* The array contained has a fixed number of rows and columns, where the columns are for */
    float table[500][8];            /* specific variables.*/
    float time;                               
 }  RSIMType;

   
char * displaytitle = "Atlas V 400 Rocket Simulation Data", * graph1yalabel = "Acceleration (m/s^2)",* graph1yblabel = "Velocity (m/s^1)", * graph1xlabel = "Time (s)", * graph2ylabel = "Rocket Mass (kg)", *graph2xlabel = "Altitude (m)";


/* Above is the declaration of the different axes titles that are displayed within the graphics window.  They can easily be changed to*/
/* fit the user's requirements.*/


int Menu() {                                            /* Function that displays the list of options to the user.*/
    int option1;
    system("cls");
    printf("\nThis is a program to simulate a rocket launch, A set of default simulation data is ready to view.\n\n");
    printf("1.\tDisplay experimental and calculated data\n");
    printf("2.\tChange parameters\n");
    printf("3.\tPlot graphs\n");
    printf("4.\tSave data to Excel\n");
    printf("5.\tEnd program\n\n");
    printf("Please select: ");
    option1 = ValidateData();          /* The validate function is called here.*/
    return option1; }                  /* The user's choice is returned to the main function.*/



int ValidateData() {                                       /* This function validates the user's input, and displays an error message*/
    int selection;                                         /* if invalid.*/
    while((scanf("%d", &selection) != 1)) {                /* The function checks for a numerical input - scanf will return a '1' if so.*/
      fflush(stdin);                                       /* The buffer is flushed so that the return key isn't registered as an input.*/
      printf("\nInvalid entry, please try again.\n\nPlease select: "); }
    return selection; }

void  DisplayDataTable(RSIMType datatable, int fromrow, int torow) {
      int row, column; 
      int LineCount, dummy;             /* Function displays list of every value, 10 rows at a time so the user isn't sent masses of data.*/

      if (fromrow < 0) {  fromrow = 0;  }
      if (torow > datatable.currentrow-1) {  torow = datatable.currentrow-1;  }  /* Don’t display empty rows.*/
      if (datatable.currentrow == 0) {  printf("The current table is empty\n");  }
      else {
          LineCount=0;  
	      printf("     Time\t\t Air Density\t\t     Drag\t\tAcceleration\t\t   Velocity\t\t   Altitude\t\t    Gravity\t\tMassofRocket\n"); 		        /* Column headings.*/              
          for (row=fromrow;  row<=torow;  row=row+1) {
	        
	           LineCount = LineCount+1;   /* Registers line count.*/
               if (LineCount==11) {
                    printf("Press RETURN to continue ...\n"); 
                    fflush(stdin);
                    dummy = getchar();
                    LineCount=1;}
               for (column=1;  column<=MAXCOL;  column=column+1) {
		            printf("%9.2f\t\t", datatable.table[row][column]);     
	           } 
	           printf("\n");
          }                   
      }
}


RSIMType ClearDataTable(RSIMType datatable) {       /* This function clears the entire current set of data.*/
     
    int  row, column;     
     
         
    for (row=0;  row<=MAXROW-1;  row=row+1) {      /* Otherwise returns to the menu.*/
        for (column=1;  column<=MAXCOL;  column=column+1) {
            datatable.table[row][column] = 0;
        }
    }
    datatable.currentrow = 0; 
           /* Finally the current row's data is set to 0.*/
    return datatable;
}

int DrawText1(int x, int y, char * message, int xAlign, int yAlign, char color) {          /*horizontal axes, labels*/                                                
    GrTextOption grt;                                                     
    grt.txo_font = &GrDefaultFont;                                        
    grt.txo_fgcolor.v = color;
    grt.txo_bgcolor.v = GrNOCOLOR;
    grt.txo_direct = GR_TEXT_RIGHT;
    grt.txo_xalign = xAlign;
    grt.txo_yalign = yAlign;
    grt.txo_chrtype = GR_BYTE_TEXT;
    GrDrawString( message,strlen( message ),x,y,&grt );
}

int DrawText2(int x, int y, char * message, int xAlign, int yAlign, char color) {             /*vertical axes, labels*/                                             
    GrTextOption grt;                                                     
    grt.txo_font = &GrDefaultFont;                                        
    grt.txo_fgcolor.v = color;
    grt.txo_bgcolor.v = GrNOCOLOR;
    grt.txo_direct = GR_TEXT_UP;
    grt.txo_xalign = xAlign;
    grt.txo_yalign = yAlign;
    grt.txo_chrtype = GR_BYTE_TEXT;
    GrDrawString( message,strlen( message ),x,y,&grt );
}     


/* Function to find the maximum value for a variable*/
float max_function(RSIMType datatable, int column){
    float max;
    int row;
    max = datatable.table[0][column];
    for (row=0;  row<MAXROW;  row=row+1){ 
        if( datatable.table[row][column] > max ) {
            max = datatable.table[row][column];
        }
    }
    return ceil(max);   
} 
      
/* A scaling function where x is the x data point from the initial equations, xres is the bottom right x coordinate of the graph you wish
 to plot on and xmax is the maximum x value in the data set*/      

int scale(int start, int end, RSIMType datatable, int row, int column, int max){ 
      return start + (end - start) * datatable.table[row][column] / max;
}


int reverse_scale(int x, int start, int end, int max){
    return ((x - start)*max)/(end-start);    
    }
  

  
 void plot_graph(int column_x, int column_y, RSIMType datatable, int xs, int ys, int xe, int ye, int color){
     float xmax, ymax;     
     int row;
     int x1,x2,y1,y2;
     xmax = max_function(datatable, column_x);
     ymax = max_function(datatable, column_y);
     for (row=0;  row<MAXROW-1;  row=row+1){
        x1 = scale(xs, xe, datatable, row, column_x, xmax);
        y1 = scale(ys, ye, datatable, row, column_y, ymax);
        x2 = scale(xs, xe, datatable, row+1, column_x, xmax);
        y2 = scale(ys, ye, datatable, row+1, column_y, ymax);
        GrLine(x1, y1, x2, y2, color);
     }             
}
      
      
/*Graph_plotter(int column_x, int column_y, RSIMType datatable,  int max, int variable_xa, int variable_xb, int variable_y1a, int variable_y2a, int variable_yb){*/
Graph_plotter( RSIMType datatable,  int max, int variable_xa, int variable_xb, int variable_y1a, int variable_y2a, int variable_yb, char y1a_label[40], char y2a_label[40]){
    int xres, yres, ob, ib, i, exit_cross_size, xa_max, y1a_max, y2a_max, xb_max, yb_max;
    int x1a, y1a, x2a, y2a, x1b, y1b, x2b, y2b;
    int row_in_table;
    char str[80];
    char temp[80];
    char displaytitle[34], time_display[40], yb_label[40], y1a_value[40], y2a_value[40], yb_value[40], xa_value[40], xb_value[40];
    GrMouseEvent evt;
    GrSetMode(GR_width_height_graphics,GetSystemMetrics(SM_CXSCREEN),GetSystemMetrics(SM_CYSCREEN)); /* makes the graphics window full size*/
    GrClearScreen(15);    /* Makes the graphics window white*/
    ob = 40;
    ib = 10;   
    xres=GrScreenX();
    yres=GrScreenY();
     
    sprintf (time_display, "Time (s)");
    sprintf (yb_label, "Acceleration (m/s^2)");
    
    x1a = ob/2;
    y1a = (yres/2)+(ob/2)-2*(ib);
    x2a = xres-(ob/2);
    y2a = (ob/2)+ib;
    
    x1b = ob/2;
    y1b = yres-(3*ib);
    x2b = (2*(xres-ob))/3;
    y2b = ((yres+20)/2 + 2*ib);
    

    GrLine(x1a, y1a, x1a, y2a, 0);
    GrLine(x1a, y1a, x2a, y1a, 0);
    GrLine(x2a, y1a, x2a, y2a, 0);
    GrLine(x1b, y1b, x1b, y2b, 0);
    GrLine(x1b, y1b, x2b, y1b, 0);
    DrawText1(xres/2, 0, displaytitle, GR_ALIGN_CENTER, GR_ALIGN_TOP, GrBlack());    /*labels*/
    DrawText2(ib, (yres/4)+((3/2)*ib), y1a_label, GR_ALIGN_CENTER, GR_ALIGN_CENTER,GrBlack());
    DrawText2(xres - ib, (yres/4)+((3/2)*ib), y2a_label, GR_ALIGN_CENTER, GR_ALIGN_CENTER,GrBlack());
    DrawText1(xres/2, ((yres/2)+(ib)), time_display, GR_ALIGN_CENTER, GR_ALIGN_TOP, GrBlack());
    DrawText2(ib, yres-(yres/4), yb_label, GR_ALIGN_CENTER, GR_ALIGN_CENTER, GrBlack());
    DrawText1(xres/3, yres-2*ib, time_display, GR_ALIGN_CENTER, GR_ALIGN_TOP, GrBlack());
    
    plot_graph(variable_xa, variable_y1a, datatable, x1a, y1a, x2a, y2a, 4); /* function to plot accleration against time on the top graph*/
    plot_graph(variable_xa, variable_y2a, datatable, x1a, y1a, x2a, y2a, 1); /* function to plot velocity against time on the top graph*/
    plot_graph(variable_xb, variable_yb, datatable, x1b, y1b, x2b, y2b, 2); /* function to plot altitude against time on the top graph*/
   
    exit_cross_size = 15;
    GrLine(xres - exit_cross_size,0,xres,exit_cross_size,12);      /*Exit cross*/
    GrLine(xres - exit_cross_size,exit_cross_size,xres,0,12);      /*Exit cross*/
    
    xa_max = max_function(datatable, variable_xa);
    y1a_max = max_function(datatable, variable_y1a);
    y2a_max = max_function(datatable, variable_y2a);
    xb_max = max_function(datatable, variable_xb);
    yb_max = max_function(datatable, variable_yb);
    
    i = 1;
    y1a_value[0] = '#';
    while(i != -1){

        GrMouseGetEventT(GR_M_LEFT_DOWN,&evt,0L);
        
        GrFilledBox(((2*(xres-ob))/3)+(ob/2),(yres/2)+(ob/2)+ib,xres-(ob/2),yres-(ob/2),14);
        GrTextXY(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(ob), y1a_label, 4, 14);            /*prints the paramenters from the graph in the 'parameter box' and displays the values of the point at which the user clicks*/
        GrTextXY(((2*(xres-ob))/3)+(ob/2)+ib,(4*yres/6)+(ob/3), y2a_label, 1, 14);
        GrTextXY(((2*(xres-ob))/3)+(ob/2)+ib,(5*yres/6)-(ob/3), yb_label, 2, 14);
        GrTextXY(((2*(xres-ob))/3)+(ob/2)+ib,(yres)-(ob), time_display, 0, 14);
                
        if(evt.buttons == 1){                                   /*this displays the values of the graph where you click in the parameters box*/
            sprintf (y1a_value, "= N/A");        
            sprintf (y2a_value, "= N/A");
            sprintf (yb_value, "= N/A");
            sprintf (xb_value, "= N/A");
            if (evt.y > y2a && evt.y < y1a && evt.x < x2a && evt.x > x1a) {         /*the if statements determine whether the user is clicking on the graph or not*/
                      sprintf (y1a_value, "= %d", reverse_scale( evt.y, y1a, y2a, y1a_max));
                      sprintf (y2a_value, "= %d", reverse_scale( evt.y, y1a, y2a, y2a_max));
                      sprintf (xb_value, "= %d", reverse_scale( evt.x, x1a, x2a, xa_max));               /*sprintf used to change the values printed in the parameter box in the graphics window*/         
            }
            if (evt.y > y2b && evt.y < y1b && evt.x < x2b && evt.x > x1b) {
                      sprintf (yb_value, "= %d", reverse_scale( evt.y, y1b, y2b, yb_max));
                      sprintf (xb_value, "= %d", reverse_scale( evt.x, x1b, x2b, xb_max));
            }             
            if (evt.x > (xres -exit_cross_size) && evt.y < exit_cross_size) {       /*this is to exit the while loop when the cross is clicked*/
               i = -1;
               GrMouseUnInit();
               GrSetMode(GR_default_text);
            }       
        }
        if(y1a_value[0] != '#'){
        GrTextXY(((2*(xres-ob))/3)+(ob/2)+ib+200,(yres/2)+(ob), y1a_value, 4, 14);            /*prints the paramenters from the graph in the 'parameter box' and displays the values of the point at which the user clicks*/
        GrTextXY(((2*(xres-ob))/3)+(ob/2)+ib+200,(4*yres/6)+(ob/3), y2a_value, 1, 14);
        GrTextXY(((2*(xres-ob))/3)+(ob/2)+ib+200,(5*yres/6)-(ob/3), yb_value, 2, 14);
        GrTextXY(((2*(xres-ob))/3)+(ob/2)+ib+200, yres-ob, xb_value, 0, 14);
        }
    }  
} 

void graph_menu(RSIMType datatable){
     int xa, xb, y1a, y2a, yb, max, choice, option2, i;
     char y1a_label[40], y2a_label[40];
             
             i = 1;
             while(i != 3){
                          printf("\nWhich two variables would you like to plot against time?.\n\n");
                          printf("1.\tRocket mass (Kg)\n");
                          printf("2.\tAltitude (m)\n");
                          printf("3.\tDrag (N)\n");
                          printf("4.\tGravity (N)\n");
                          printf("5.\tAir Density (Kg/m^3\n");
                          printf("6.\tVelocity (m/s)\n\n");
                          choice = ValidateData();
                          switch(choice){
                                    case 1: if (i == 1) {
                                               y1a = 8;  
                                               sprintf (y1a_label, "Rocket Mass (kg)");
                                               }
                                            if (i == 2) {
                                               y2a = 8;  
                                               sprintf (y2a_label, "Rocket Mass (kg)");
                                            }
                                            break;                 
                                    case 2: if (i == 1) {
                                               y1a = 6;
                                               sprintf (y1a_label, "Altitude (m)");
                                            }
                                            if (i == 2) {
                                               y2a = 6;
                                               sprintf (y2a_label, "Altitude (m)");
                                            }
                                            break;                                                                                       
                                    case 3: if (i == 1) {
                                               y1a = 3; 
                                               sprintf(y1a_label, "Drag (N)");
                                            }
                                            if (i == 2) {
                                               y2a = 3;
                                               sprintf (y2a_label, "Drag (N)");
                                            }
                                            break;    
                                    case 4: if (i == 1) {
                                               y1a = 7; 
                                               sprintf (y1a_label, "Gravity (N)");
                                            }
                                            if (i == 2) {
                                               y2a = 7; 
                                               sprintf (y2a_label, "Gravity (N)");
                                            }
                                            break;    
                                    case 5: 
                                         if (i == 1) {
                                            y1a = 2; 
                                            sprintf (y1a_label, "Air Density (kg/M^3)");
                                         }
                                            if (i == 2) {
                                               y2a = 2; 
                                               sprintf (y2a_label, "Air Density (kg/M^3)");
                                            }
                                            break;
                                    case 6: if (i == 1) {
                                               y1a = 5;
                                               sprintf(y1a_label, "Velocity (m/s)");
                                            }
                                            if (i == 2) {
                                               y2a = 5; 
                                               sprintf (y2a_label, "Velocity (m/s)");
                                            }
                                            break;
                          }
                          i = i + 1;
             }                       
             Graph_plotter(datatable, max, 1, 1, y1a, y2a, 4, y1a_label, y2a_label);
}

float timer(float dt){
      time = (time + dt);
      return time;
}  

float calc_thrust(float *thrust, int *number_of_boosters, float gravity, float fuel_rate_of_solid_rocket_boosters, float fuel_rate_of_atlas_booster, float dt, float *thrust_percentage, float SRB_burn_time, float atlas_booster_burn_time) {
      if (time <= SRB_burn_time) {
           *thrust = (*number_of_boosters * 1688400 + 3827000) * *thrust_percentage;
      } else {
           if (time <= atlas_booster_burn_time) {
                    *thrust = 3827000 * *thrust_percentage;
           } else { 
              *thrust = 0;
           }
      }
}

float calc_density(int *start_temp, float molar_mass) {
      float pressure = pressure_at_sea_level * exp((-1 * molar_mass * gravity * altitude) / (gas_constant * *start_temp));
      density = (pressure * molar_mass)/(gas_constant * *start_temp);
      return density;
      }

float calc_drag(float *drag_coefficient, float area_which_experiences_drag) {
      drag = *drag_coefficient * density * (pow(velocity, 2) / 2) * area_which_experiences_drag;
      return drag;
      }

float calc_acceleration(float thrust) {
      acceleration = ((thrust - drag) / mass) - gravity;
      return acceleration;
      }                          
           
float calc_velocity(float dt) {
     velocity = velocity + (acceleration * dt);
     return velocity;
     }

float calc_altitude(float dt) {
      altitude = altitude + velocity * dt + ((acceleration * pow(dt, 2)) / 2);
      return altitude;
      }

float calc_gravity() {
      gravity = (gravitational_constant * mass_of_earth) / (pow(radius_of_earth + altitude, 2));
      return gravity;
      }

float calc_mass(float fuel_rate_of_solid_rocket_boosters, float fuel_rate_of_atlas_booster, float dt, int *number_of_boosters, int *detach_SRB_time, int *inert_mass) {
      if (time == *detach_SRB_time) { mass = mass - (*number_of_boosters * 5740); }
      if (time <= 94) {mass = mass - ((fuel_rate_of_solid_rocket_boosters * *number_of_boosters) + fuel_rate_of_atlas_booster) * dt;}
      else mass = mass - fuel_rate_of_atlas_booster * dt;
      if ( mass < *inert_mass) {mass = *inert_mass; }
      return mass;
}





RSIMType AddData(RSIMType datatable, int *number_of_boosters, int *payload, int *inert_mass, float *drag_coefficient, float *thrust_percentage, int *start_temp, int *detach_SRB_time, int *detach_atlas_booster_time) {

     float thrust, total_time, area_which_experiences_drag, molar_mass, dt, fuel_rate_of_solid_rocket_boosters, fuel_rate_of_atlas_booster, SRB_burn_time, atlas_boster_burn_time;

     total_time = *detach_atlas_booster_time; 
     area_which_experiences_drag = (PI / 4) * pow(5.4, 2);
     molar_mass = 0.02897;
     dt = 1;
     fuel_rate_of_solid_rocket_boosters = (1688400 * *thrust_percentage) / (9.81 * 279.3);
     fuel_rate_of_atlas_booster = (3827000 * *thrust_percentage) / (9.81 * 311.3);
     SRB_burn_time = 40939 / fuel_rate_of_solid_rocket_boosters;
     atlas_boster_burn_time = 284089 / fuel_rate_of_atlas_booster;
     
     gravity = 9.819296623;
     velocity = 0;
     altitude = 0;  
     mass = *payload + (*number_of_boosters * 46697) + *inert_mass + 284089;
     time = -1;

     
           /* Function takes data input and stores it in the data structure, row by row.*/
     if (datatable.currentrow >= (MAXROW)) {
    	   printf("The array of data is full"); }
     else {
           while (time < total_time) { 
                    datatable.table[datatable.currentrow][COLt] = timer(dt);  
	                calc_thrust(&thrust, number_of_boosters, gravity, fuel_rate_of_solid_rocket_boosters, fuel_rate_of_atlas_booster, dt, thrust_percentage, SRB_burn_time, atlas_boster_burn_time);
                    datatable.table[datatable.currentrow][COLRo] = calc_density(start_temp, molar_mass);  
	                datatable.table[datatable.currentrow][COLdrag] = calc_drag(drag_coefficient, area_which_experiences_drag);
	                datatable.table[datatable.currentrow][COLa] = calc_acceleration(thrust);
	                datatable.table[datatable.currentrow][COLv] = calc_velocity(dt);
	                datatable.table[datatable.currentrow][COLh] = calc_altitude(dt);
	                datatable.table[datatable.currentrow][COLg] = calc_gravity();
	                datatable.table[datatable.currentrow][COLm] = calc_mass(fuel_rate_of_solid_rocket_boosters,fuel_rate_of_atlas_booster, dt, number_of_boosters, detach_SRB_time, inert_mass);
	                datatable.currentrow = datatable.currentrow + 1;
                    }                                               
              return datatable; }
    }   


float ChangeParameters(int *payload, int *number_of_boosters, int *inert_mass, int *centaur_engine_type, int *start_temp, float *drag_coefficient, float *thrust_percentage, int *detach_SRB_time, int *detach_atlas_booster_time){
      int  inputfinished, i, ii;
      char userinput[500];
      inputfinished = 0;
	  while (inputfinished == 0) {
            system("cls");
            printf("\nPlease select a parameter you would like to change(Press 9 to return to the main menu):\n\n");
            printf("1 = Payload (Default = medium)\n2 = Number of Boosters (Default = 3)\n3 = Centaur Engine Type (Default = single)\n4 = Temperature (Default = 280)\n5 = Drag Coefficient (Default = 0.42)\n6 = Thrust percentage (Default = 72%)\n7 = Detach Solid Rocket Booster Time (Default = 115)\n8 = Detach Atlas Booster (Default = 250)\n9 = Return to Menu\n");
            printf("Please select: ");
            fflush(stdin);
		    fgets(userinput, sizeof(userinput), stdin);
            system("cls");
            switch(userinput[0]) {                              /* 'toupper' converts all inputs to uppercase - lowercase can then be used too - user's choice.*/
                    case '1': MassOfPayload(payload, i); break;           /* Case statement with the different options to the user.*/
                    case '2': NumberOfBoosters(number_of_boosters); printf("%d", *number_of_boosters); break;                                                       
                    case '3': CentaurEngineType(centaur_engine_type, inert_mass); break;           
                    case '4': temperature(start_temp); break;
                    case '5': DecideDragCofficient(drag_coefficient);printf("working"); break;
                    case '6': DecideThrustPercentage(thrust_percentage); break;
                    case '7': DetachSRBTime(detach_SRB_time); break;
                    case '8': DetachABTime(detach_atlas_booster_time); break;
                    case '9': inputfinished = 1; break;
                    default: printf("\nInvalid option, try again\n"); 
           }   
      } 
    
}      
    

int MassOfPayload(int *payload, int i) {
      printf ("\nPlease select the payload which you would like to use:\n\n");
      printf("1. Short pay load\n2. Medium pay load\n3. Long pay load\n");
      printf("Please select: ");
      i = ValidateData();
      if (i == 1){ 
            *payload = 3524;}
      else if (i == 2){
            *payload = 4003;} 
      else if (i == 3){ 
            *payload = 4379;}
      else { printf("\nInvalid entry, please try again\n"); }
      }

int NumberOfBoosters(int *number_of_boosters) {
      *number_of_boosters = -1;
      while (*number_of_boosters < 0 || *number_of_boosters > 5) {
                 printf ("\nPlease enter the number of Solid ROcket Boosters you would like to use (You can have 0-5): ");
                 *number_of_boosters = ValidateData();
      }
}

int CentaurEngineType(int *centaur_engine_type, int *inert_mass) {
      int i;
      printf ("\nPlease select the type of common centaur engine you would like to use:\n\n");
      printf("1. Single engine centaur\n2. Duel engine centaur\n");
      printf("Please select: ");
      i = ValidateData();
      if (i == 1) {
            *centaur_engine_type = 99200;
            *inert_mass = 51203; }
      else if (i == 2) {
            *centaur_engine_type = 198400;
            *inert_mass = 51418; }
      else { printf("\nInvalid entry, please try again\n"); }
      }
      
int temperature(int *start_temp) {
    printf ("\nPlease enter the temperature you would like to launch the rocket at (in Kelvin): ");
    *start_temp = ValidateData();
    }

int DecideDragCofficient(float *drag_coefficient) {
      printf ("\nPlease enter the drag coefficient you would like to use: ");
      *drag_coefficient = ValidateData();
}

int DecideThrustPercentage(float *thrust_percentage) {
      printf ("\nPlease enter the percentage of the thrust you would like to use (a higher percentage will use the fuel up quicker): ");
      *thrust_percentage = ValidateData() / 100;
      printf ("the Solid Rocket Boosters with run out of fuel after %fseconds\nThe Atlas booster will run out of fuel after %fseconds", 40939 / (1688400 * *thrust_percentage) / (9.81 * 279.3), 284089 / ((3827000 * *thrust_percentage) / (9.81 * 311.3)));
      }

int DetachSRBTime(int *detach_SRB_time) {
    printf ("\nPlease enter the time after launch you would like the Solid Rocket Boosters to detach: ");
    *detach_SRB_time = ValidateData();
}

int DetachABTime(int *detach_atlas_booster_time) {
    printf ("\nPlease enter the time after launch you would like the the Atlas Booster to detach (This will be the time at the the simulation ends): ");
    *detach_atlas_booster_time = ValidateData();
}

int Export_to_excel(RSIMType datatable, int *detach_atlas_booster_time) {
    char spreadsheet_location[1000], filename[50];
    int j;
    FILE *spreadsheet;
    getcwd(spreadsheet_location, sizeof(spreadsheet_location));
    sprintf(filename, "Atlas V simulation.csv");
    spreadsheet = fopen(filename, "w+");
    fprintf(spreadsheet, "Time,Density,Drag,Acceleration,Velocity,Altitude,Gravity,Mass\n");
    for(j=0;j<*detach_atlas_booster_time;j++){
                          fprintf(spreadsheet,"%f,", datatable.table[j][COLt]);
                          fprintf(spreadsheet,"%f,", datatable.table[j][COLRo]);
                          fprintf(spreadsheet,"%f,", datatable.table[j][COLdrag]);
                          fprintf(spreadsheet,"%f,", datatable.table[j][COLa]);
                          fprintf(spreadsheet,"%f,", datatable.table[j][COLv]);
                          fprintf(spreadsheet,"%f,", datatable.table[j][COLh]);
                          fprintf(spreadsheet,"%f,", datatable.table[j][COLg]);
                          fprintf(spreadsheet,"%f\n", datatable.table[j][COLm]);       
                }
                fclose(spreadsheet);
    printf("Your spreadsheet has been saved to:\n'%s'\n\n", spreadsheet_location);
    system("PAUSE");
    system("cls");
    }

RSIMType AskChangeParameters(RSIMType datatable, int *number_of_boosters, int *payload, int *inert_mass, float *drag_coefficient, float *thrust_percentage, int *start_temp, int *detach_SRB_time, int *detach_atlas_booster_time, int *centaur_engine_type) {
    char confirmclear;
    confirmclear = 'D';                        /* Causes the program to enter the while loop below.*/
     
     if (datatable.currentrow == 0) {
         confirmclear = ' '; }
	 while (confirmclear != 'Y' && confirmclear != 'y' && confirmclear != 'N' && confirmclear != 'n') {  /* Allows for both uppercase and lowercase.*/
	        printf("The current table contains data which will be lost. Are you sure you want to clear and change the current table, (Y/N)?");
	        scanf(" %c", &confirmclear);       /* Asks the user if they are sure they want to clear the table.*/
            printf("\n");
            if (confirmclear == 'Y' || confirmclear == 'y') {  /* If the choice is yes, then all the row's values are set to 0.*/
                datatable = ClearDataTable(datatable);
                ChangeParameters(payload, number_of_boosters, inert_mass, centaur_engine_type, start_temp, drag_coefficient, thrust_percentage, detach_SRB_time, detach_atlas_booster_time);
                datatable = AddData(datatable, number_of_boosters, payload, inert_mass, drag_coefficient, thrust_percentage, start_temp, detach_SRB_time, detach_atlas_booster_time);   
            }
    }
 
 return datatable;   
}

int main() {                                                            /* Main function.*/
    RSIMType datatable;
    int choice;
    int i;  
    int xborderless;
    int yborderless;  
    int column_x;
    int column_y;
    int payload = 4003;                                                 /*medium payload is the default*/
    int centaur_engine_type = 99200;                                            /*this is the thrust from single engine common centaur, this is the default, not used but added for extendability*/
    float drag_coefficient = 0.42;
    float thrust_percentage = 0.72;
    int start_temp = 280;
    int detach_SRB_time = 115;
    int detach_atlas_booster_time = 250;         
    int number_of_boosters = 3;                                      /* Variables declared, and current row in the data table set to 0.*/
    int inert_mass = 51203;
    int max = 0;      
    datatable.currentrow = 0;
    choice = 99; 
    datatable = AddData(datatable, &number_of_boosters, &payload, &inert_mass, &drag_coefficient, &thrust_percentage, &start_temp, &detach_SRB_time, &detach_atlas_booster_time);                                                       /* Enters the while loop.*/
    while(choice != 0) {
        choice = Menu();                                                /* Case statements corresponding to the users choice.*/
        switch(choice) {                                                /* Each one calls a function to perform a certain task.*/
                case 1:  DisplayDataTable(datatable, 0, MAXROW); break;                                               /* When a function like this is called - the data structure is*/
                case 2:  datatable = AskChangeParameters(datatable, &number_of_boosters, &payload, &inert_mass, &drag_coefficient, &thrust_percentage, &start_temp, &detach_SRB_time, &detach_atlas_booster_time, &centaur_engine_type); break;                             /* The break stops the while loop from running through each option*/                                                             
                case 3:  /*Graph_plotter(column_x, column_y, datatable, max, 1, 6, 4, 5, 8);*/graph_menu(datatable); break;
                case 4:  Export_to_excel(datatable, &detach_atlas_booster_time); break;
                case 5:  choice = 0; break;                               /* once a case has been selected.*/   
                default: printf("\nInvalid entry, please try again\n"); } 
   }
   printf("Program ended.\n");
   return 0; 
}
