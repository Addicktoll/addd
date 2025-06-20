/*
//mainxaml
<Window x:Class="DEMO1.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:DEMO1"
        mc:Ignorable="d"
        Title="Главное окно" Height="450" Width="800" Icon="C:\Users\addicktoll\Desktop\DEMO1\main.ico">
    <ScrollViewer VerticalScrollBarVisibility="Auto" DockPanel.Dock="Top">
        <Grid>
            <DockPanel LastChildFill="True">
                <Button Content="Добавить партнера" Click="BtnAddPartner_Click" DockPanel.Dock="Top" HorizontalAlignment="Left" Margin="10" Width="150" Height="42" Background="#67BA80" />
                <Border BorderBrush="Black" BorderThickness="1" Margin="0,5" Padding="10">
                    <ItemsControl ItemsSource="{Binding DataItems}">
                        <ItemsControl.ItemTemplate>
                            <DataTemplate>
                                <Border BorderBrush="Black" BorderThickness="1" Margin="0,5" Padding="10" MouseLeftButtonDown="PartnerItem_MouseDoubleClick">
                                    <StackPanel>
                                        <TextBlock>
                                    <Run Text="{Binding partners_type}" FontSize="16"/>
                                    <Run Text="|" FontSize="16"/>
                                    <Run Text="{Binding partners_name}" FontSize="16"/>
                                        </TextBlock>
                                        <TextBlock Text="{Binding director}" />
                                        <TextBlock>
                                    <Run Text="+7"/>
                                    <Run Text="{Binding telephone}"/>
                                        </TextBlock>
                                        <TextBlock>
                                    <Run Text="Рейтинг: "/>
                                    <Run Text="{Binding rate}"/>
                                        </TextBlock>
                                    </StackPanel>
                                </Border>
                            </DataTemplate>
                        </ItemsControl.ItemTemplate>
                    </ItemsControl>
                </Border>
            </DockPanel>
        </Grid>
    </ScrollViewer>
</Window>
//maincs
using System.Windows;
using Npgsql;
using System.Collections.ObjectModel;
using System.Windows.Controls;

namespace DEMO1
{
    ///
<summary>
    /// Interaction logic for MainWindow.xaml
    ///</summary>
public partial class MainWindow : Window
    {
        public ObservableCollection
<Partner>DataItems { get; set; } = new ObservableCollection
    <Partner>();

        public MainWindow()
        {
            InitializeComponent();
            DataContext = this; // Устанавливаем DataContext
            LoadData();
        }

        public class Partner
        {
            public string partners_type { get; set; }
            public string partners_name { get; set; }
            public string director { get; set; }
            public string email { get; set; }
            public string telephone { get; set; }
            public string address { get; set; }
            public string inn { get; set; }
            public int rate { get; set; }
        }

        private void LoadData()
        {
            string connectionString = "Host=localhost;Port=5432;Database=postgres;Username=postgres;Password=123456789;SearchPath=ex6";

            try
            {
                // Очищаем коллекцию перед загрузкой новых данных
                DataItems.Clear();
                using (var conn = new NpgsqlConnection(connectionString))
                {
                    conn.Open();

                    using (var cmd = new NpgsqlCommand("SELECT * FROM partners", conn))
                    using (var reader = cmd.ExecuteReader())
                    {
                        while (reader.Read())
                        {
                            var partner = new Partner
                            {
                                partners_type = reader.IsDBNull(1) ? string.Empty : reader.GetString(1),
                                partners_name = reader.IsDBNull(2) ? string.Empty : reader.GetString(2),
                                director = reader.IsDBNull(3) ? string.Empty : reader.GetString(3),
                                email = reader.IsDBNull(4) ? string.Empty : reader.GetString(4),
                                telephone = reader.IsDBNull(5) ? string.Empty : reader.GetString(5),
                                address = reader.IsDBNull(6) ? string.Empty : reader.GetString(6),
                                inn = reader.IsDBNull(7) ? string.Empty : reader.GetString(7),
                                rate = reader.IsDBNull(8) ? 0 : reader.GetInt32(8)
                            };
                            DataItems.Add(partner);
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка подключения к базе данных: " + ex.Message);
            }
        }
        private void BtnAddPartner_Click(object sender, RoutedEventArgs e)
        {
            var addPartnerWindow = new AddPartnerWindow();
            addPartnerWindow.Owner = this;
            addPartnerWindow.ShowDialog();

            // Обновляем список партнеров после закрытия окна добавления
            LoadData();
        }
        private void PartnerItem_MouseDoubleClick(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            // Проверяем, был ли это двойной клик
            if (e.ClickCount == 2)
            {
                // Получаем выбранного партнера
                var border = sender as Border;
                var partner = border.DataContext as Partner;

                // Открываем окно редактирования
                var editPartnerWindow = new EditPartnerWindow(partner);
                editPartnerWindow.Owner = this;
                editPartnerWindow.ShowDialog();

                // Обновляем список партнеров после закрытия окна редактирования
                LoadData();
            }
        }
    }
}
//addxaml
        <Window x:Class="DEMO1.AddPartnerWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Добавить партнера" Height="550" Width="800">
            <Grid Margin="10">
                <StackPanel>
                    <TextBlock Text="Тип партнера:" />
                    <ComboBox x:Name="cmbPartnersType" Margin="0,5,0,10" />

                    <TextBlock Text="Наименование партнера:" />
                    <TextBox x:Name="txtPartnersName" Margin="0,5,0,10" />

                    <TextBlock Text="Директор:" />
                    <TextBox x:Name="txtDirector" Margin="0,5,0,10" />

                    <TextBlock Text="Электронная почта партнера:" />
                    <TextBox x:Name="txtEmail" Margin="0,5,0,10" />

                    <TextBlock Text="Телефон:" />
                    <TextBox x:Name="txtTelephone" Margin="0,5,0,10" />

                    <TextBlock Text="Адресс партнера:" />
                    <TextBox x:Name="txtAddress" Margin="0,5,0,10" />

                    <TextBlock Text="ИНН партнера:" />
                    <TextBox x:Name="txtInn" Margin="0,5,0,10" />

                    <TextBlock Text="Рейтинг:" />
                    <TextBox x:Name="txtRate" Margin="0,5,0,10" />

                    <StackPanel Orientation="Horizontal" HorizontalAlignment="Right" Margin="0,10">
                        <Button Content="Сохранить" Click="BtnAdd_Click" Margin="0,0,10,0" Padding="10,5" Background="#67BA80" />
                        <Button Content="Отмена" Click="Cancel_Click" Padding="10,5" Background="#FFFFFF" />
                    </StackPanel>
                </StackPanel>
            </Grid>
        </Window>
        //addcs
using System;
using System.Collections.Generic;
using System.Windows;
using Npgsql;

namespace DEMO1
{
    public partial class AddPartnerWindow : Window
    {
        public AddPartnerWindow()
        {
            InitializeComponent();
            LoadPartnerTypes();
        }

        private void LoadPartnerTypes()
        {
            string connectionString = "Host=localhost;Port=5432;Database=postgres;Username=postgres;Password=123456789;SearchPath=ex6";

            try
            {
                using (var conn = new NpgsqlConnection(connectionString))
                {
                    conn.Open();
                    string query = "SELECT DISTINCT partners_type FROM partners";

                    using (var cmd = new NpgsqlCommand(query, conn))
                    using (var reader = cmd.ExecuteReader())
                    {
                        var partnerTypes = new List
        <string>();
                        while (reader.Read())
                        {
                            partnerTypes.Add(reader.GetString(0));
                        }
                        cmbPartnersType.ItemsSource = partnerTypes;
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка при загрузке типов партнеров: " + ex.Message);
            }
        }

        private void BtnAdd_Click(object sender, RoutedEventArgs e)
        {
            string connectionString = "Host=localhost;Port=5432;Database=postgres;Username=postgres;Password=123456789;SearchPath=ex6";

            try
            {
                using (var conn = new NpgsqlConnection(connectionString))
                {
                    conn.Open();
                    string query = "INSERT INTO partners (partners_type, partners_name, director, email, address, inn, telephone, rate) " +
                                  "VALUES (@type, @name, @director, @email, @address, @inn, @telephone, @rate)";

                    using (var cmd = new NpgsqlCommand(query, conn))
                    {
                        cmd.Parameters.AddWithValue("type", cmbPartnersType.Text);
                        cmd.Parameters.AddWithValue("name", txtPartnersName.Text);
                        cmd.Parameters.AddWithValue("director", txtDirector.Text);
                        cmd.Parameters.AddWithValue("email", txtEmail.Text);
                        cmd.Parameters.AddWithValue("telephone", txtTelephone.Text);
                        cmd.Parameters.AddWithValue("address", txtAddress.Text);
                        cmd.Parameters.AddWithValue("inn", txtInn.Text);
                        cmd.Parameters.AddWithValue("rate", txtRate.Text); // Без преобразования типов

                        cmd.ExecuteNonQuery();
                    }
                }
                MessageBox.Show("Партнер успешно добавлен!");
                this.Close();
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка при добавлении партнера: " + ex.Message);
            }
        }

        private void Cancel_Click(object sender, RoutedEventArgs e)
        {
            Close();
        }
    }
}
//editxaml
            <Window x:Class="DEMO1.EditPartnerWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Редактировать партнера" Height="550" Width="800">
                <Grid Margin="10">
                    <StackPanel>
                        <TextBlock Text="Тип партнера:" />
                        <ComboBox x:Name="cmbPartnersType" Margin="0,5,0,10" />

                        <TextBlock Text="Наименование партнера:" />
                        <TextBox x:Name="txtPartnersName" Margin="0,5,0,10" />

                        <TextBlock Text="Директор:" />
                        <TextBox x:Name="txtDirector" Margin="0,5,0,10" />

                        <TextBlock Text="Электронная почта партнера:" />
                        <TextBox x:Name="txtEmail" Margin="0,5,0,10" />

                        <TextBlock Text="Телефон:" />
                        <TextBox x:Name="txtTelephone" Margin="0,5,0,10" />

                        <TextBlock Text="Адрес партнера:" />
                        <TextBox x:Name="txtAddress" Margin="0,5,0,10" />

                        <TextBlock Text="ИНН партнера:" />
                        <TextBox x:Name="txtInn" Margin="0,5,0,10" />

                        <TextBlock Text="Рейтинг:" />
                        <TextBox x:Name="txtRate" Margin="0,5,0,10" />

                        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right" Margin="0,10">
                            <Button Content="Сохранить" Click="BtnSave_Click" Margin="0,0,10,0" Padding="10,5" Background="#67BA80" />
                            <Button Content="Отмена" Click="BtnCancel_Click" Margin="0,0,10,0" Padding="10,5" Background="#FFFFFF" />
                            <Button Content="Удалить" Click="DeleteButton_Click" Padding="10,5" Background ="Red"/>
                        </StackPanel>
                    </StackPanel>
                </Grid>
            </Window>
            //editcs
using System;
using System.Collections.Generic;
using System.Windows;
using Npgsql;
using static DEMO1.MainWindow;

namespace DEMO1
{
    public partial class EditPartnerWindow : Window
    {
        private readonly Partner _partner;

        public EditPartnerWindow(Partner partner)
        {
            InitializeComponent();
            _partner = partner;
            LoadPartnerTypes();
            FillFields();
        }

        private void FillFields()
        {
            cmbPartnersType.Text = _partner.partners_type;
            txtPartnersName.Text = _partner.partners_name;
            txtDirector.Text = _partner.director;
            txtEmail.Text = _partner.email;
            txtTelephone.Text = _partner.telephone;
            txtAddress.Text = _partner.address;
            txtInn.Text = _partner.inn;
            txtRate.Text = _partner.rate.ToString();
        }

        private void LoadPartnerTypes()
        {
            string connectionString = "Host=localhost;Port=5432;Database=postgres;Username=postgres;Password=123456789;SearchPath=ex6";

            try
            {
                using (var conn = new NpgsqlConnection(connectionString))
                {
                    conn.Open();
                    string query = "SELECT DISTINCT partners_type FROM partners";

                    using (var cmd = new NpgsqlCommand(query, conn))
                    using (var reader = cmd.ExecuteReader())
                    {
                        var partnerTypes = new List
            <string>();
                        while (reader.Read())
                        {
                            partnerTypes.Add(reader.GetString(0));
                        }
                        cmbPartnersType.ItemsSource = partnerTypes;
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка при загрузке типов партнеров: " + ex.Message);
            }
        }

        private void BtnSave_Click(object sender, RoutedEventArgs e)
        {
            string connectionString = "Host=localhost;Port=5432;Database=postgres;Username=postgres;Password=123456789;SearchPath=ex6";

            try
            {
                using (var conn = new NpgsqlConnection(connectionString))
                {
                    conn.Open();
                    string query = "UPDATE partners SET " +
                                   "partners_type = @type, " +
                                   "partners_name = @name, " +
                                   "director = @director, " +
                                   "email = @email, " +
                                   "address = @address, " +
                                   "inn = @inn, " +
                                   "telephone = @telephone, " +
                                   "rate = @rate " +
                                   "WHERE partners_name = @originalName";

                    using (var cmd = new NpgsqlCommand(query, conn))
                    {
                        cmd.Parameters.AddWithValue("type", cmbPartnersType.Text);
                        cmd.Parameters.AddWithValue("name", txtPartnersName.Text);
                        cmd.Parameters.AddWithValue("director", txtDirector.Text);
                        cmd.Parameters.AddWithValue("email", txtEmail.Text);
                        cmd.Parameters.AddWithValue("telephone", txtTelephone.Text);
                        cmd.Parameters.AddWithValue("address", txtAddress.Text);
                        cmd.Parameters.AddWithValue("inn", txtInn.Text);
                        cmd.Parameters.AddWithValue("rate", txtRate.Text);
                        cmd.Parameters.AddWithValue("originalName", _partner.partners_name);

                        cmd.ExecuteNonQuery();
                    }
                }
                MessageBox.Show("Данные партнера успешно обновлены!");
                this.Close();
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка при обновлении данных: " + ex.Message);
            }
        }

        private void DeleteButton_Click(object sender, RoutedEventArgs e)
        {
            var result = MessageBox.Show("Вы уверены, что хотите удалить этого партнера?",
                "Подтверждение", MessageBoxButton.YesNo);

            if (result != MessageBoxResult.Yes)
                return;

            string connectionString = "Host=localhost;Port=5432;Database=postgres;Username=postgres;Password=123456789;SearchPath=ex6";

            try
            {
                using (var conn = new NpgsqlConnection(connectionString))
                {
                    conn.Open();
                    string query = "DELETE FROM partners WHERE partners_name = @name";

                    using (var cmd = new NpgsqlCommand(query, conn))
                    {
                        cmd.Parameters.AddWithValue("name", _partner.partners_name);
                        cmd.ExecuteNonQuery();
                    }
                }
                MessageBox.Show("Партнер успешно удален!");
                this.Close();
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка при удалении: " + ex.Message);
            }
        }

        private void BtnCancel_Click(object sender, RoutedEventArgs e)
        {
            Close();
        }
    }
}
