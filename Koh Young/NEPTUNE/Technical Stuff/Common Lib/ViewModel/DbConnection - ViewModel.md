### Constructor
```csharp
public DbConnectionViewViewModel(string ip, string id, string password, EventHandler inspResultUpdater, bool isConnected = false)
{

	// Get information for db connect and store to this view model properties
	this.IP = ip;
	this.Password = password;
	this.ID = id;

	this.MaximumnCount = 20; // Maximum number of PCB to query
	
	this.IndicatorContent = "Loading..."; // Set loading text

	// Set date time
	this.StartDateTime = DateTime.Now.AddMonths(-7).Date.Add(new TimeSpan(24, 00, 00));
	this.EndDateTime = DateTime.Now.Date.Add(new TimeSpan(23, 59, 59));

	// InpItems source for second grid
	this.InpItems = new List<InspectionItem>();

	// Instance of data parser
	this.resultDataParser = new CciResultParser();

	// Instance of data reader
	this.resultDataReader = new ResultDbReader();
	this.resultDataReader.SetConnection(ID, Password, IP);
	
	this.IsConnected = isConnected; // Default to false everytime the constructor is called 

	this.boardInfos = new Dictionary<Guid?, (BoardInfo, List<InspectionItem>)>(); // See more in [[DbConnection - Models]]

	// Source for IP and IP ComboBox
	Servers = new ObservableCollection<ComboInfo>()
	{
		new ComboInfo("127.0.0.1","Localhost"),
		new ComboInfo("10.3.11.12","Unit1"),
		new ComboInfo("10.3.11.225","Unit2"),
	};
	SelectedItem = Servers.FirstOrDefault();

	SearchTypes = new ObservableCollection<ComboInfo>()
	{
		new ComboInfo("BARCODE", "BARCODE"),
		new ComboInfo("PCBGUID", "PCB GUID"),
		new ComboInfo("PCBID", "PCB ID"),
	};
	SelectedSearchType = SearchTypes.LastOrDefault();

	this.inspResultUpdater = inspResultUpdater;

	// Set Execute method for each DelegateCommand
	this.ConnectCommand = new DelegateCommand(async () => await TryConnectDB());
	this.PCBSelectionChangedCommand = new DelegateCommand<object>(async pcb => await Showindicator(SelectedPCBChanged(pcb)));
	this.InspItemSelectionChangedCommand = new DelegateCommand<object>(OnSelectedItemChanged);
	this.PcbGridRefreshCommand = new DelegateCommand(async () => await Showindicator(RefreshQuery()));
	this.ResultSelectionChangedCommand = new DelegateCommand<object>(SelectedResultChanged);
	this.SearchCommand = new DelegateCommand(async () => await Showindicator(OnSearchExecute()));
}
```

---

### Switch between search type
```xml
<ComboBox
	x:Name="searchCombo"
	HorizontalAlignment="Right"
	Margin="0 0 5 0"
	SelectedIndex="0">
	<ComboBoxItem Content="Period" />
	<ComboBoxItem Content="Search" />
</ComboBox>
```
- Then in each stack panel (search, period):
```xml
<StackPanel.Style >
	<Style TargetType="StackPanel">
		<Style.Triggers>
			<DataTrigger Binding="{Binding SelectedIndex, ElementName=searchCombo}" Value="1"> // Or 0 for period search
				<Setter Property="Visibility" Value="Collapsed" />
			</DataTrigger>
		</Style.Triggers>
	</Style>
</StackPanel.Style>
```