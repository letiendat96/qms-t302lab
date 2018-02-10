# qms-t302lab
Just repository
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Drawing.Printing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Data.SqlClient;
using System.IO;
using System.Drawing.Drawing2D;
using System.Drawing.Text;


namespace QMS
{

    public partial class Request : Form
    {
        private SqlConnection con;
        private DataTable dtCustomer = new DataTable("tblCustomer");

        private DataTable dtIdCurrent = new DataTable("tblIdCurrent");

        private SqlDataAdapter da = new SqlDataAdapter();
        private SqlDataAdapter ds = new SqlDataAdapter();

        private Bitmap bmp;
        private int numCustomer = 0;
        private string timeTicket;

        private Font printFont;
        private Font fontNumber;
        private StreamReader streamToPrint, streamtoPrint2;
       
        private void Connect()
        {            
            String cn = "Data Source=ADMIN-PC\\SQLEXPRESS;Initial Catalog=qms;Integrated Security=True";
            try
            {
                con = new SqlConnection(cn);
                con.Open();
            }
            catch (Exception e)
            {
                MessageBox.Show("Khong the ket noi den du lieu ", "Error", MessageBoxButtons.OK);
                Console.WriteLine("\n Message Connection----\n", e.Message);
            }

        }

        #region  Ngat ket noi voi Server

        private void InsertNumCustomer(int num) {
            System.IO.FileStream fs = new System.IO.FileStream(@"..\test.txt", FileMode.Create, FileAccess.Write, FileShare.None);
            StreamWriter wr = new StreamWriter(fs);
            string[] str = { "Số thứ tự:  " + num + "\n", "Quý khách vui lòng chờ !", "Time: " + timeTicket };
            for (int i = 0; i <= 2; i++){
                    wr.WriteLine(str[i]);
            }
                // giai phong va dong tep
                wr.Flush();
                wr.Close();
                fs.Close();

        }

        private void Disconnect()
        {
            con.Close();
            con.Dispose();
            con = null;
        }

        #endregion

        public Request()
        {
            InitializeComponent();
            timer1.Start();
        }

        private void GetData(){
             SqlCommand command = new SqlCommand();
             command.Connection = con;
             command.CommandType = CommandType.Text;
             command.CommandText = @"select * from dbo.Counter_Customer";
             da.SelectCommand = command;
             da.Fill(dtCustomer);
         }
   
        private int RandomService (int min, int max){
            Random random = new Random();
            return random.Next(min, max);
        }
        // Lay ID khach hang gan nhat

        private void GetIdCurrent() {
            SqlCommand command = new SqlCommand();
            command.Connection = con;
            command.CommandType = CommandType.Text;
            command.CommandText = @"select * from dbo.CustomerCurrent";
            ds.SelectCommand = command;
            ds.Fill(dtIdCurrent);
            //Doc du lieu tu dbo.CustomerCurrent
            foreach (DataRow dr in dtIdCurrent.Rows) {
                numCustomer = Convert.ToInt16(dr["IdCurrent"].ToString());            
            }
        }

        private void btnRequest1_Click(object sender, EventArgs e)
         {
            
             numCustomer = numCustomer + 1;
             Writefile_Content(numCustomer);
             //--------------------------------------------------------------
             // Chen So quay va Khach hang
             DataRow row = dtCustomer.NewRow();
             //Lay du lieu
             row["SCounter"] =  Convert.ToString(RandomService(1,4));
             row["IdCustomer"] = Convert.ToString(numCustomer);
             dtCustomer.Rows.Add(row);
            
             //Insert Data
             SqlCommand commandInsert = new SqlCommand();
             commandInsert.Connection = con;
             commandInsert.CommandType = CommandType.Text;
             commandInsert.CommandText = @"insert into dbo.Counter_Customer (SCounter, IdCustomer) values (@SCounter, @IdCustomer)";

             //Chen so quay
             commandInsert.Parameters.Add("@SCounter", SqlDbType.NVarChar, 10, "SCounter");
             commandInsert.Parameters.Add("@IdCustomer", SqlDbType.Int, 2, "IdCustomer");

             da.InsertCommand = commandInsert;
             da.Update(dtCustomer);

             //--------------------------------------------------------------
             // In phieu khach hang
             try
             {
                 //Mo hop thoai PrintPreview
                 //PrintDialog print = new PrintDialog();
                 //print.ShowDialog();
                 streamToPrint = new StreamReader(@"..\test.txt");
                 try
                 {
                     printFont = new Font("Arial", 10, FontStyle.Regular);
                     fontNumber = new Font("Arial", 24, FontStyle.Bold);
                     PrintDocument pdBitmap = new PrintDocument();
                     pdBitmap.PrintPage += new PrintPageEventHandler(this.PrintBitmap);
                     pdBitmap.PrintController = new StandardPrintController(); // Xoa hop thoai Printing dialog
                     pdBitmap.Print();

                     PrintDocument pdContent = new PrintDocument();
                     pdContent.PrintPage += new PrintPageEventHandler(this.PrintContent);
                     pdContent.PrintController = new StandardPrintController(); // Xoa hop thoai Printing dialog
                     pdContent.Print();
                 }
                 finally {
                     streamToPrint.Close();  
                 }                 
             }
             catch (Exception ex) {
                 MessageBox.Show(ex.Message);
             }

             //Form1 _Form1 = new Form1();            `
             //_Form1.Show();
             //Hide();
         }

        private void Request_Load(object sender, EventArgs e)
         {
             Connect();
             GetData();
             GetIdCurrent();
         }

        private void btnRequest2_Click(object sender, EventArgs e)
         {
             
         }

        private void PrintBitmap(object sender, PrintPageEventArgs e)
         {

             try
             {
                 bmp = new Bitmap(@"F:\XHTD\Bank2.png", true);
             }
             catch (IOException ex) {
                 MessageBox.Show(ex.Message);
             }

             #region Custom Bitmap
             //Bitmap temp = new Bitmap(bmp.Width, bmp.Height);

             //RectangleF rectf = new RectangleF(10, 150,240, 37 );

             //Graphics g = Graphics.FromImage(temp);
             //g.SmoothingMode = SmoothingMode.AntiAlias;
             //// The interpolation mode determines how intermediate values between two endpoints are calculated.
             //g.InterpolationMode = InterpolationMode.HighQualityBicubic;

             //// Use this property to specify either higher quality, slower rendering, or lower quality, faster rendering of the contents of this Graphics object.
             //g.PixelOffsetMode = PixelOffsetMode.HighQuality;
             //// This one is important
             //g.TextRenderingHint = TextRenderingHint.AntiAliasGridFit;

             //StringFormat format = new StringFormat()
             //{
             //    Alignment = StringAlignment.Center, 
             //    LineAlignment = StringAlignment.Center
 
             //};
             //g.DrawString("1000", new Font("Tahoma", 10),Brushes.Black, rectf, format);
             //g.Flush();
           
             //Graphics mg = Graphics.FromImage(bmp);
             //mg.CopyFromScreen(this.Location.X, this.Location.Y, 0, 0, this.Size);
             #endregion

             e.Graphics.DrawImage(bmp, 1, 1);
          }

        private void PrintContent(object sender, PrintPageEventArgs e) {
            float linesPerPage = 0;
            float yPos = 0;
            int count = 0;
            float leftMargin = 0;
            float topMargin = 0;
            string line = null;

            linesPerPage = e.MarginBounds.Height / printFont.GetHeight(e.Graphics);
            while (count < linesPerPage && ((line = streamToPrint.ReadLine()) != null)){
                yPos = topMargin + (count * printFont.GetHeight(e.Graphics));
                // In IsCustomer
                if (count == 0)
                {
                    if (numCustomer < 10) {
                        e.Graphics.DrawString(line, fontNumber, Brushes.Black, 15 *5 , yPos, new StringFormat());
                    }
                    else if (numCustomer >= 10 && numCustomer < 100) {
                        e.Graphics.DrawString(line, fontNumber, Brushes.Black, 15 *4, yPos, new StringFormat());
                    }
                    else if (numCustomer >= 100 && numCustomer < 1000) {
                        e.Graphics.DrawString(line, fontNumber, Brushes.Black, 15 * 4, yPos, new StringFormat());
                    }else if(numCustomer >= 1000 && numCustomer <10000){
                        e.Graphics.DrawString(line, fontNumber, Brushes.Black, 15 * 3, yPos, new StringFormat());
                    }
                    
                }
                //In Content
                else {
                    e.Graphics.DrawString(line, printFont, Brushes.Black, leftMargin, yPos, new StringFormat());
                }
                count++;
            }
            if (line != null) {
                e.HasMorePages = true;
            }else{
                e.HasMorePages = false;
            } 
        
        }
        
        #region Ghi noi dung phieu khach hang
        public void Writefile_Content(int numID) {
            System.IO.FileStream fs = new System.IO.FileStream(@"..\test.txt", FileMode.Create, FileAccess.Write, FileShare.None);
            StreamWriter wr = new StreamWriter(fs);
            //---------------------------------------------------
            string[] str = { numID + "\n" , "\n  GIAO DỊCH KHÁCH HÀNG" + "\n", "Quý khách vui lòng chờ!\nSố phiếu của quý khách sẽ \nđược gọi khi đến lượt.",
                             "\nPlease seat and wait!\nYour number will be called \nin your turn.", "\nNgày:" + timeTicket };
            for (int i = 0; i <= 4; i++)
            {
                wr.WriteLine(str[i]);
            }
            //Giai phong va dong tep
            wr.Flush();
            wr.Close();
            fs.Close();

        }
        #endregion

        // Lay thoi gian 
        private void timer1_Tick(object sender, EventArgs e)
        {
            DateTime tn = DateTime.Now;
            timeTicket = tn.ToString("dd/MM/yyyy HH:mm:ss");
        }

        // Ghi so phieu cua khach hang xuong CustomerCurrent
        private void Request_FormClosing(object sender, FormClosingEventArgs e)
        {
            if (numCustomer > 0) {
                DataRow row = dtIdCurrent.Select("IdConstant = " + Convert.ToInt32("1"))[0];
                row.BeginEdit();
                row["IdCurrent"] = numCustomer.ToString(); 
                row.EndEdit();

                SqlCommand commandUpdate = new SqlCommand();
                commandUpdate.Connection = con;
                commandUpdate.CommandType = CommandType.Text;
                commandUpdate.CommandText = @"Update dbo.CustomerCurrent set IdCurrent = @IdCurrent where IdConstant = @IdConstant";

                commandUpdate.Parameters.Add("@IdConstant", SqlDbType.Int, 2, "IdConstant");
                commandUpdate.Parameters.Add("@IdCurrent", SqlDbType.Int, 2, "IdCurrent");

                ds.UpdateCommand = commandUpdate;
                ds.Update(dtIdCurrent);
                MessageBox.Show("Da ghi du lieu numberID", "Thong bao", MessageBoxButtons.OK);
                dtIdCurrent.Clear();
                GetIdCurrent();


            }

        }
    }
}
