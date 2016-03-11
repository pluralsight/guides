Sometimes it's handy to have the same data in two different enviroments:
- In SQL for joins and filters
- In C# so the numbers make sense to us humans and to contain the number of possible choices.

<pre>
<#@ template debug="false" hostSpecific="true" language="C#" #>
<#@ assembly name="System.Data" #>
<#@ import namespace="System.Data" #>
<#@ import namespace="System.Data.SqlClient" #>
<#@ import namespace="System.Text.RegularExpressions" #>
<#@ import namespace="System.Collections.Generic" #>
<#@ import namespace="System.Configuration" #>
<#@ import namespace="System.IO" #>
<#@ output extension=".cs" #>
<#

string path = Path.GetDirectoryName(Host.TemplateFile);
// Be sure to include default database name, if it exists
string constr = @"Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=" + path + @"\App_Data\exampleDB.mdf;Integrated Security=True;Connect Timeout=30";

List<Entity> entities = new List<Entity>();

entities.Add(new Entity() {EnumNameSpace = "model.Enums",
								EnumName="ThisEnum",
								EnumType = "int",
								TableSchema="dbo", 
								TableName="tbl", 
								TableTextColumn="txt", 
								TableValueColumn="id",
								WhereCondition=""});

this.WriteLine("//Do NOT CHANGE THIS FILE - Auto generated code from AutoEnums.tt template");
this.WriteLine("//To generate this file, open .tt file and save it");

//Geração dos enums
//ToDo: se tiverem o mesmo schema, declarar esse schema envolvendo todos os enums que lhe pertencem
SqlConnection con = new SqlConnection(constr);
con.Open();

foreach (Entity entity in entities)
{
	SqlCommand cmd = new SqlCommand(String.Format("SELECT {0}, {1} FROM {2}.{3} WHERE {0} IS NOT NULL AND {1} IS NOT NULL {4} ORDER BY {1}", entity.TableTextColumn, entity.TableValueColumn,entity.TableSchema, entity.TableName, entity.WhereCondition),con);
	SqlDataReader rdr = cmd.ExecuteReader();
#>
namespace <#= entity.EnumNameSpace #> {

	/// &lt;summary&gt;
	/// Namespace    : <#= entity.EnumNameSpace #>
	/// Name         : <#= entity.EnumName #>
	/// Type         : <#= entity.EnumType #>
	/// Table        : <#= entity.TableSchema #>.<#= entity.TableName #>
	/// Value column : <#= entity.TableValueColumn #>
	/// Text column  : <#= entity.TableTextColumn #>
    /// &lt;/summary&gt;
	public enum <#= Entity.ParseEntityField(entity.EnumName) + (entity.EnumType == "int" ? "" : " : " + entity.EnumType ) #> 
	{
<#
PushIndent("	");
PushIndent("	");
if (rdr.Read())
{
	bool loop = true;
	while (loop)
	{
		string opt = string.Format("{0} = {1}", Entity.ParseEntityField(rdr[entity.TableTextColumn].ToString()), rdr[entity.TableValueColumn].ToString());
		Write(opt);
		loop = rdr.Read();
		if (loop)
		{
			WriteLine(",");
		}
		else
		{
			WriteLine("");
		}
	
	}
}
PopIndent();
PopIndent();
#>
	}
}
<#
	rdr.Close();
}
con.Close();
#>
<#+ 
//Classes (takes a + befour the #)
	public class Entity
    {
        public string EnumNameSpace;
        public string EnumName;
		public string EnumType;
        public string TableSchema;
        public string TableName;
        public string TableTextColumn;
        public string TableValueColumn;
	    public string WhereCondition;

	    //If the text field that gives the text for the enum itens has invalid characters, this removes them
        public static string ParseEntityField(string s)
        {
            // Remove the dot, left bracket, right bracket, space
            // and slash characters from the fieldname
			// Don't cover all cases, but worked for me
            string pattern = "[\\.\\[\\]\\s/-]*";
            Regex reg = new Regex(pattern, RegexOptions.None);
		    string replaced = reg.Replace(s, String.Empty);
            return System.Text.Encoding.UTF8.GetString(System.Text.Encoding.GetEncoding("iso-8859-8").GetBytes(replaced));
        }
    }
#>
</pre>

Code in https://github.com/correiadefreitas/SQL_T4_Enum