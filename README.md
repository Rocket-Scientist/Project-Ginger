Project-Ginger
==============
#include <stdio.h>          /* The libraries needed for the code below.*/
#include <string.h>
#include <grx20.h>
 
const int   MAXROW = 1000, MAXCOL = 6, COLt = 1, COLg = 2, COLRo = 3, COLa = 4, COLv = 5, COLh = 6;	


typedef struct	RSIMStructure {     /* Defining the data structure that is used throughout the program.*/
    float table[1000][6];            /* The array contained has a fixed number of rows and columns, where the columns are for */
    int	currentrow;                 /* specific variables.*/                  
 }  RSIMType;


int Menu();
int ValidateData();
RSIMType ClearDataTable(RSIMType datatable);
void DisplayDataTable(RSIMType datatable, int fromrow, int torow);     

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
                case 5:  printf("Task performed\n");break;
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
	      printf("ROW\t\tTime\t\tGravity\t\tAir Density\t\tAcceleration\t\tVelocity\t\tAltitude\n"); 		        /* Column headings.*/
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




