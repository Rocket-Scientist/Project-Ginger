Project-Ginger
==============
#include <stdio.h>          /* The libraries needed for the code below.*/
#include <string.h>
#include <grx20.h>
#define pressure_at_sea_level 101325, gravitational_constant 6.67384*pow(10, -11), radius_of_earth 6378100, mass_of_earth 5.9726*pow(10, 24), boltzman_constant 1.3806488 * pow(10, -23)

 
const int   MAXROW = 1000, MAXCOL = 8, COLt = 1, COLRo = 2, COLdrag = 3, COLa = 4, COLv = 5, COLh = 6, COLg = 7, COLm = 8;	


typedef struct	RSIMStructure {     /* Defining the data structure that is used throughout the program.*/
    float table[1000][8];            /* The array contained has a fixed number of rows and columns, where the columns are for */
    int	currentrow;                 /* specific variables.*/                  
 }  RSIMType;


int Menu();
int ValidateData(), DrawText(int x, int y, char * graphtitle, int xAlign, int yAlign);
RSIMType ClearDataTable(RSIMType datatable);
void DisplayDataTable(RSIMType datatable, int fromrow, int torow), Graph_plotter();     
char * displaytitle = "Atlas V 400 Rocket Simulation Data", * graph1ylabel = "Acceleration (m/s^2)", * graph1xlabel = "Time (s)", * graph2ylabel = "Altitude (m)", * graph2xlabel = "Fuel Usage (kg)";

/* Above is the declaration of the functions used within the program, along with the default axis labels within global character arrays.*/






int main() {                                                            /* Main function.*/
    RSIMType datatable;
    int choice;                                                         /* Variables declared, and current row in the data table set to 0.*/
    datatable.currentrow = 0;
    choice = 99;                                                        /* Enters the while loop.*/
    while(choice != 0) {
        choice = Menu();                                                /* Case statements corresponding to the users choice.*/
        switch(choice) {                                                /* Each one calls a function to perform a certain task.*/
                case 1:  printf("Task performed\n");break;          /* If the input is incorrect, an error message will be displayed.*/
                case 2:  DisplayDataTable(datatable, 0, MAXROW); break;                                               /* When a function like this is called - the data structure is*/
                case 3:  ClearDataTable(datatable); break;
                case 4:  printf("Task performed\n");break;                             /* The break stops the while loop from running through each option*/                                                             
                case 5:  Graph_plotter();break;
                case 6:  choice = 0; break;                               /* once a case has been selected.*/   
                default: printf("\nInvalid entry, please try again\n"); } 
   }
   printf("Program ended.\n");
   return 0; }



int Menu() {                                            /* Function that displays the list of options to the user.*/
    int option1;
    printf("\n\n\n");
    printf("1.\tAdd data.\n");
    printf("2.\tDisplay experimental and calculated data.\n");
    printf("3.\tClear data set in table format\n");
    printf("4.\tUse default data\n");
    printf("5.\tPlot graphs\n");
    printf("6.\tEnd program\n\n");
    printf("Please select: ");
    option1 = ValidateData();          /* The validate function is called here.*/
    return option1; }                    /* The user's choice is returned to the main function.*/



int ValidateData() {                                       /* This function validates the user's input, and displays an error message*/
    int selection;                                          /* if invalid.*/
    while((scanf("%d", &selection) != 1)) {                 /* The function checks for a numerical input - scanf will return a '1' if so.*/
      fflush(stdin);                                        /* The buffer is flushed so that the return key isn't registered as an input.*/
      printf("\nInvalid entry, please try again.\n\nPlease select: "); }
    return selection; }

void  DisplayDataTable(RSIMType datatable, int fromrow, int torow) {
      int row, column;                  /* Function displays list of every data value - not just latest 10.*/

      if (fromrow < 0) {  fromrow = 0;  }
      if (torow > datatable.currentrow-1) {  torow = datatable.currentrow-1;  }  /* Don’t display empty rows.*/
      if (datatable.currentrow == 0) {  printf("The current table is empty\n");  }
      else {
	      printf("ROW\t\tTime\t\tAir Density\t\tDrag\t\tAcceleration\t\tVelocity\t\tAltitude\t\tGravity\t\tMass of Rocket\n"); 		        /* Column headings.*/
	      for (row=fromrow;  row<=torow;  row=row+1) {
	           printf("%d\t\t", row+1);
               for (column=1;  column<=MAXCOL;  column=column+1) {
		            printf("%.2f\t\t", datatable.table[row][column]);     
	           } 
          }                   
     }
}

RSIMType ClearDataTable(RSIMType datatable) {       /* This function clears the entire current set of data.*/
     char confirmclear;
     int  row, column;
     confirmclear = 'D';                        /* Causes the program to enter the while loop below.*/
     
     if (datatable.currentrow == 0) {
         confirmclear = ' '; }
	 while (confirmclear != 'Y' && confirmclear != 'y' && confirmclear != 'N' && confirmclear != 'n') {  /* Allows for both uppercase and lowercase.*/
	        printf("The current table contains data which will be lost. Are you sure you want to clear the current table, (Y/N)?");
	        scanf(" %c", &confirmclear);       /* Asks the user if they are sure they want to clear the table.*/
            printf("\n"); }
         if (confirmclear == 'Y' || confirmclear == 'y') {  /* If the choice is yes, then all the row's values are set to 0.*/
	         for (row=0;  row<=MAXROW-1;  row=row+1) {      /* Otherwise returns to the menu.*/
		          for (column=1;  column<=MAXCOL;  column=column+1) {
		               datatable.table[row][column] = 0; }
	         }
         datatable.currentrow = 0; }  /* Finally the current row's data is set to 0.*/
      return datatable; }

      
void Graph_plotter() {
    int xres, yres, ob, ib;
    char str[80];
    char temp[80];
    GrMouseEvent evt;
    ob = 40;
    ib = 10;                                    /* plotting graphs, axes and labels*/
    GrSetMode(GR_width_height_graphics,1024,600);

    xres=GrScreenX();
    yres=GrScreenY();

    GrLine(ob/2,(ob/2)+ib,ob/2,(yres/2),15);
    GrLine(ob/2,(yres/2)+(ob/2)-2*(ib),xres-(ob/2),(yres/2),15);
    GrLine(ob/2,((yres+20)/2 + 2*ib),ob/2,yres-(3*ib),15);
    GrLine(ob/2,yres-(3*ib),(2*(xres-ob))/3,yres-(3*ib),15);
    DrawText1(xres/2, 0, displaytitle, GR_ALIGN_CENTER, GR_ALIGN_TOP );    /*labels*/
    DrawText2(ib, (yres/4)+((3/2)*ib), graph1ylabel, GR_ALIGN_CENTER, GR_ALIGN_CENTER );
    DrawText1(xres/2, ((yres/2)+(ib)), graph1xlabel, GR_ALIGN_CENTER, GR_ALIGN_TOP );
    DrawText2(ib, yres-(yres/4), graph2ylabel, GR_ALIGN_CENTER, GR_ALIGN_CENTER );
    DrawText1(xres/3, yres-2*ib, graph2xlabel, GR_ALIGN_CENTER, GR_ALIGN_TOP );
    
    GrFilledBox(((2*(xres-ob))/3)+(ob/2),(yres/2)+(ob/2)+ib,xres-(ob/2),yres-(ob/2),8);
    DrawText1(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(ob), graph1ylabel, GR_ALIGN_LEFT, GR_ALIGN_TOP);
    DrawText1(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(2*ob), graph1ylabel, GR_ALIGN_LEFT, GR_ALIGN_TOP);
    DrawText1(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(3*ob), graph1ylabel, GR_ALIGN_LEFT, GR_ALIGN_TOP);
    DrawText1(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(4*ob), graph1ylabel, GR_ALIGN_LEFT, GR_ALIGN_TOP);
    
    /* GrContext *GrCurrentContext(void) or GetScreenContext()         */
    /*while(1){

        GrMouseGetEventT(GR_M_EVENT,&evt,0L);
        
        if(evt.flags & (GR_M_KEYPRESS | GR_M_BUTTON_CHANGE))
        {
            
            strcpy(str,"GrScreenX ");
            sprintf(temp, "%d", GrScreenX());
            strcat(str,temp);
            strcat(str," GrScreenY ");
            sprintf(temp, "%d", GrScreenY());
            strcat(str,temp);
            DrawText1(100,100,str,GR_ALIGN_CENTER, GR_ALIGN_CENTER);
        }
        
    }*/
    
}




int DrawText1(int x, int y, char * message, int xAlign, int yAlign) {          /*horizontal axes, labels*/                                                
    GrTextOption grt;                                                     
    grt.txo_font = &GrDefaultFont;                                        
    grt.txo_fgcolor.v = GrWhite();
    grt.txo_bgcolor.v = GrBlack();
    grt.txo_direct = GR_TEXT_RIGHT;
    grt.txo_xalign = xAlign;
    grt.txo_yalign = yAlign;
    grt.txo_chrtype = GR_BYTE_TEXT;
    GrDrawString( message,strlen( message ),x,y,&grt ); }     
    
    
    
    
int DrawText2(int x, int y, char * message, int xAlign, int yAlign) {             /*vertical axes, labels*/                                             
    GrTextOption grt;                                                     
    grt.txo_font = &GrDefaultFont;                                        
    grt.txo_fgcolor.v = GrWhite();
    grt.txo_bgcolor.v = GrBlack();
    grt.txo_direct = GR_TEXT_UP;
    grt.txo_xalign = xAlign;
    grt.txo_yalign = yAlign;
    grt.txo_chrtype = GR_BYTE_TEXT;
    GrDrawString( message,strlen( message ),x,y,&grt ); }     
    
float scalex(float xnew, float xres, float xmax){ 
      xnew = x*(xres/xmax);
      return xnew;}
/* A sclaing function where x is the x data point from the initial equations, xres is the bottom right x coordinate of the graph you wish to plot on and xmax is the maximum x value in the data set*/

float scaley(float ynew, float yres, float ymax){ 
      ynew = y*(yres/ymax);
      return ynew;}
/* A sclaing function where y is the y data point from the initial equations, yres is the bottom right y coordinate of the graph you wish to plot on and ymax is the maximum y value in the data set*/

float max_function(int n){
      xmax = x[0];
      ymax = y[0];

      for (i=0;i<n;i=i+1)
      {
      if( x[i] > xmax ) xmax = x[i];
      if( y[i] > ymax ) ymax = y[i];
      }
}    

/* 3 functions above not defined at the start*/
