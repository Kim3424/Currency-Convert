using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Xml.Linq;
using System.Xml;
using System.Net.Http;

namespace WindowsFormsApp1
{
    public partial class Form1 : Form
    {
        private const string API_URL = "https://openexchangerates.org/api/latest.json";
        private const string API_KEY = "YOUR_API_KEY_HERE";
        public Form1()
        {
            InitializeComponent();
        }
        private async void Form1_Load(object sender, EventArgs e)
        {
            XmlDocument xml = new XmlDocument();
            xml.Load("http://www.vietcombank.com.vn/exchangerates/ExrateXML.aspx");

            XmlNodeList noXml;
            noXml = xml.SelectNodes("/ExrateList/Exrate");
            int i = 0;
            for (i = 0; i <= noXml.Count - 1; i++)
            {
                ListViewItem item = new ListViewItem(noXml.Item(i).Attributes["CurrencyCode"].InnerText);
                item.SubItems.Add(noXml.Item(i).Attributes["CurrencyName"].InnerText);
                item.SubItems.Add(noXml.Item(i).Attributes["Buy"].InnerText);
                item.SubItems.Add(noXml.Item(i).Attributes["Transfer"].InnerText);
                item.SubItems.Add(noXml.Item(i).Attributes["Sell"].InnerText);
                liv_DanhSach.Items.Add(item);


            }
        }

        private async Task LoadCurrencies()
        {
            try
            {
                // Lấy danh sách tỷ giá từ API
                JObject data = await GetExchangeRates();
                if (data != null)
                {
                    // Thêm các loại tiền tệ vào ComboBox
                    foreach (var rate in data["rates"])
                    {
                        string currency = rate.Path.Split('.')[1];
                        comboBoxFrom.Items.Add(currency);
                        comboBoxTo.Items.Add(currency);
                    }

                    // Đặt mặc định giá trị đầu tiên
                    comboBoxFrom.SelectedIndex = 0;
                    comboBoxTo.SelectedIndex = 1;
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error loading currencies: {ex.Message}");
            }
        }
        private void button5_Click(object sender, EventArgs e)
        {
            if (double.TryParse(txtAmount.Text, out double amount) &&
                comboBoxFrom.SelectedItem != null &&
                comboBoxTo.SelectedItem != null)
            {
                string fromCurrency = comboBoxFrom.SelectedItem.ToString();
                string toCurrency = comboBoxTo.SelectedItem.ToString();

                // Lấy tỷ giá
                JObject data = await GetExchangeRates();
                if (data != null)
                {
                    double fromRate = data["rates"][fromCurrency]?.Value<double>() ?? 1.0;
                    double toRate = data["rates"][toCurrency]?.Value<double>() ?? 1.0;

                    // Tính toán kết quả
                    double result = (amount / fromRate) * toRate;
                    resultLabel.Text = $"{amount} {fromCurrency} = {result:F2} {toCurrency}";
                }
            }
            else
            {
                MessageBox.Show("Please enter valid input!");
            }
        }
        private async Task<JObject> GetExchangeRates()
        {
            using (HttpClient client = new HttpClient())
            {
                try
                {
                    string url = $"{API_URL}?app_id={API_KEY}";
                    HttpResponseMessage response = await client.GetAsync(url);
                    response.EnsureSuccessStatusCode();
                    string responseBody = await response.Content.ReadAsStringAsync();
                    return JObject.Parse(responseBody);
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"Error fetching rates: {ex.Message}");
                    return null;
                }
            }
        }

        private void btnDel_Click(object sender, EventArgs e)
        {
            txtAmount.Clear();
            comboBoxFrom.SelectedIndex = -1; // Bỏ chọn mục trong ComboBox
            comboBoxTo.SelectedIndex = -1;

            // Xóa kết quả hiển thị
            resultLabel.Text = string.Empty;
        }
    }
}
