/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package daviscahtech_bookmanagement.Controllers;

import com.itextpdf.text.BaseColor;
import com.itextpdf.text.Document;
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.Element;
import com.itextpdf.text.Font;
import com.itextpdf.text.PageSize;
import com.itextpdf.text.Paragraph;
import com.itextpdf.text.Phrase;
import com.itextpdf.text.pdf.PdfPCell;
import com.itextpdf.text.pdf.PdfPTable;
import com.itextpdf.text.pdf.PdfWriter;
import daviscahtech_bookmanagement.Pojo.HeaderFooterPageEvent;
import daviscahtech_bookmanagement.dao.DatabaseConnection;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.net.URL;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.time.LocalDate;
import java.util.Optional;
import java.util.ResourceBundle;
import java.util.logging.Level;
import java.util.logging.Logger;
import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.concurrent.Task;
import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.Alert;
import javafx.scene.control.ButtonType;
import javafx.scene.control.ComboBox;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.layout.HBox;
import javafx.stage.FileChooser;
import javafx.stage.Window;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

/**
 * FXML Controller class
 *
 * @author davis nyandiri
 */
public class ReportPrintingController implements Initializable {



	//create an image that will be displayed in the Alert dialog box or in the controlfx notification
    Image pdfImage = new Image(getClass().getResource("/daviscahtech_bookmanagement/Resources/pdf.png").toExternalForm());
    ImageView pdfImageView = new ImageView(pdfImage);
	
    Image thumbsImage = new Image(getClass().getResource("/daviscahtech_bookmanagement/Resources/thumbs.png").toExternalForm());
    ImageView imageSuccess = new ImageView(thumbsImage);
    
    Image bookImage = new Image(getClass().getResource("/daviscahtech_bookmanagement/Resources/book.png").toExternalForm());
    ImageView bookImageView = new ImageView(bookImage);
	
     //Database variable to be used in jdbc connection
    Connection con = null;
    PreparedStatement pstmt;
    ResultSet rs;
    Statement st;
	
    //variables to hold school info, this will be helpful if you plan to generate various reports
	// containing same letter-head details: school information is stored in the database
	
    String schoolName = null;
    String schoolContact = null;
    String schoolAddress = null;
    String schoolRegion = null;
    String schoolEmail = null;
    String schoolWebsite = null;  
    
              
     //instance of FIlechooser
    private FileChooser fileChooser;
    //instance of File
    private File file;
    private Window Stage;
	

    /**
     * Initializes the controller class.
     */
    @Override
    public void initialize(URL url, ResourceBundle rb) {
        // TODO
        //connect to the database: DatabaseConnection is a CLASS that initializes connection to mysql database locally
		
        con = DatabaseConnection.connectDb();
       
        // call method to load variables with school info to initialize the decralared varaiables above
        loadSchoolInfo();
        

    }
	
    //method to automatically set school information from the database
    private void loadSchoolInfo() {
    
        try {
                 // Use sql to rettrieve data                              
                String sql3 = "SELECT * FROM schoolInfo ";
                pstmt = con.prepareStatement(sql3);

                ResultSet rs3=pstmt.executeQuery();
                if (rs3.next()) {
                     schoolName = rs3.getString("name");
                     schoolContact = rs3.getString("contact");
                     schoolAddress = rs3.getString("address");
                     schoolRegion = rs3.getString("region");
                     schoolEmail = rs3.getString("email");
                     schoolWebsite = rs3.getString("website");
                } 
                
                pstmt.close();
            
        } catch (SQLException e) {
        }
               
                //////////////////////////////////// 
    }
	
	
	// this method is linked to the button in the user interface that will trigger report generation when clicked
    @FXML
    private void generateAllSchooltBooks(ActionEvent event) throws SQLException {
        
			// the Alert dialog promts the user to confirm if they want to generate report of school books
            Alert alert = new Alert(Alert.AlertType.CONFIRMATION);
            alert.setTitle("CONFIRMATION");
            alert.setGraphic(bookImageView);
            alert.setHeaderText(null);
            alert.setContentText("Do you want to generate all school books\n");         
            Optional <ButtonType> obt = alert.showAndWait();

            if (obt.get()== ButtonType.OK) {
                // this opener allows the user to choose a location folder where to store the generated report on the machine
                 fileChooserOpener();
                   fileChooser.setTitle("Save all school books");
				   
                   //single File selection
                   file = fileChooser.showSaveDialog(Stage);                   
                    if (file != null) {
                        generalBookPleaseWait.setVisible(true);                       
                        String path = file.getAbsolutePath();
						
						//call method to generate the book report
                        createAllSchoolBooks(path);
                    }
					
                    Alert errorIssue = new Alert(Alert.AlertType.ERROR);
                    errorIssue.setTitle("Success");
                    errorIssue.setGraphic(pdfImageView);
                    errorIssue.setHeaderText(null);
                    errorIssue.setContentText("You have successfully generated All school books and it's ready for printing.\n");
                    errorIssue.showAndWait();
            
            }
    }
	
    
	            //Method to open a fileChooser dialog box
	public void fileChooserOpener(){
	                fileChooser = new FileChooser();
	                fileChooser.getExtensionFilters().addAll(                
	                new FileChooser.ExtensionFilter("PDF Files","*.pdf")
                
	        );   
	    }
        
	// this methods adds some metadata details to already generated pdf report	
   private  void addMetaData(Document document) {
	        document.addTitle( "Book watch" );
	        document.addSubject( "Books management software" );
	        document.addKeywords( "School, software, Books, Students, Teachers" );
	        document.addAuthor( "Daviscah Tech Ltd" );
	        document.addCreator( "Daviscah Tech Ltd" );
	    } 
    
	    //Method to create an empty line in the document
	     private static void addEmptyLine(Paragraph paragraph, int number) {
	        for ( int i = 0 ; i < number; i++) {
	              paragraph.add( new Paragraph( " " ));
	        }
	    }
     
	    // method to format table cell with data
	     private void insertCell(PdfPTable table,String text,int align,int colspan,Font font){
     
	         PdfPCell cell = new PdfPCell(new Phrase(text.trim(),font));
	         cell.setHorizontalAlignment(align);
	         cell.setColspan(colspan);
	         if (text.trim().equalsIgnoreCase("")) {
	             cell.setMinimumHeight(10f);
	         }
	         table.addCell(cell);
	     }
		 
	     // method to generate all school books stored in the database table called book////////////////////////////////////////////
	     public void createAllSchoolBooks(String filename) {
        
	         // create a pdf
	         Document document = new Document(PageSize.A4,5,5,20,40);
       
	          try {
            
	               String sq = "SELECT * FROM book ";
	               pstmt = con.prepareStatement(sq);
	               rs=pstmt.executeQuery();
            
				   // fonts nature to be used in the pdf document
	              Font bfBold12 = new Font(Font.FontFamily.TIMES_ROMAN, 11, Font.BOLD, BaseColor.BLACK);
	              Font bfBold = new Font(Font.FontFamily.TIMES_ROMAN, 11);
	              Font titleFont = new Font(Font.FontFamily.TIMES_ROMAN, 10);
            
	              PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(filename));
	              HeaderFooterPageEvent event = new HeaderFooterPageEvent();
	              writer.setPageEvent(event);
            
	              document.open();
	              addMetaData(document);
            
	              //Add school info to be added on top of the page
	              Paragraph intro = new Paragraph(""+schoolName+"\n"
	                      + ""+schoolAddress+" "+ schoolRegion+"\n"
	                      + "Website: "+ schoolWebsite+"\n"
	                      + "All school books as at "+ LocalDate.now() +"\n\r",bfBold12);
	              intro.setAlignment(Element.ALIGN_CENTER);            
	              document.add(intro);
            
	              // Add a table with six columns each with a specified width
	              float[] columnWidths = {3.3f,1.1f,2.1f,2.8f,1.3f,1f};
	              PdfPTable issuedBookTable = new PdfPTable(columnWidths);
	              issuedBookTable.setWidthPercentage(90f); 
				             
                  //add column titles to each    
	              insertCell(issuedBookTable, "Book ID", Element.ALIGN_LEFT, 1, bfBold12);
	              insertCell(issuedBookTable, "Class", Element.ALIGN_LEFT, 1, bfBold12);
	              insertCell(issuedBookTable, "Subject", Element.ALIGN_LEFT, 1, bfBold12);
	              insertCell(issuedBookTable, "Title", Element.ALIGN_LEFT, 1, bfBold12);
	              insertCell(issuedBookTable, "Publisher", Element.ALIGN_LEFT, 1, bfBold12);
	              insertCell(issuedBookTable, "Issued?", Element.ALIGN_LEFT, 1, bfBold12);
            
	              while (rs.next()) {
	                  String bookID = rs.getString("book_id");
	                  insertCell(issuedBookTable, bookID, Element.ALIGN_LEFT, 1, bfBold);                               
                
	                  String bookClass = rs.getString("book_class");
	                  insertCell(issuedBookTable, bookClass, Element.ALIGN_LEFT, 1, bfBold);
                
	                  String subject = rs.getString("book_category");
	                  insertCell(issuedBookTable, subject, Element.ALIGN_LEFT, 1, bfBold);
                
	                  String title = rs.getString("book_title");
	                  insertCell(issuedBookTable, title, Element.ALIGN_LEFT, 1, titleFont);
                
	                  String publisher = rs.getString("book_publisher");
	                  insertCell(issuedBookTable, publisher, Element.ALIGN_LEFT, 1, bfBold);
                
	                  boolean isAvail = rs.getBoolean("book_isAvail");
	                  String available = null;
	                  if (isAvail) {
	                      available = "No";
	                  } else {
	                      available = "Yes";
	                  }
	                  insertCell(issuedBookTable, available, Element.ALIGN_LEFT, 1, bfBold);
                         
	              }
            
	              document.add(issuedBookTable);
            
	              document.close();
	              pstmt.close();
            
            
	          } catch (DocumentException | FileNotFoundException ex) {
	              Logger.getLogger(ReportPrintingController.class.getName()).log(Level.SEVERE, null, ex);
	          }   catch (SQLException ex) {     
	                  Logger.getLogger(ReportPrintingController.class.getName()).log(Level.SEVERE, null, ex);
	              }     
     
	       }
    // method to format table cell with data
     private void insertCell(PdfPTable table,String text,int align,int colspan,Font font){
     
         PdfPCell cell = new PdfPCell(new Phrase(text.trim(),font));
         cell.setHorizontalAlignment(align);
         cell.setColspan(colspan);
         if (text.trim().equalsIgnoreCase("")) {
             cell.setMinimumHeight(10f);
         }
         table.addCell(cell);
     }
	












}