# Sweety_application

package sweety_Tests;

import org.testng.annotations.Test;
import java.text.DateFormat;
import java.text.Format;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.Date;
import java.util.List;
import org.openqa.selenium.Alert;
import org.openqa.selenium.By;
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;

public class Sweety {
	private static final Object[] String = null;
	String url = "https://damp-sands-8561.herokuapp.com";
	String successSignInMsg = "Signed in successfully.";
	String successEntryMsg = "Entry was successfully created.";
	String errorMessage = "Maximum entries reached for this date.";
	String infoMessage = "Only 4 entries allowed per day";
	String entryPage = "/entries";
	String reportPage = "/report";
	int bloodGlucoseReadings[] = { 7, 8, 9, 6 };
	int bloodGlucose1[] = { 7, 8, 9, 6, 5 };
	int lowestValue = 0;
	int highestValue = 0;
	int highestReadingdaily = 0;
	int lowestReadingdaily = 0;
	WebDriver driver = new FirefoxDriver();
	WebDriverWait wait = new WebDriverWait(driver, 30);
	String currentDate = "";
	String ModifiedDate = "";
	int totalReadings = 0;
	int totalReadingsForMoth = 0;
	String userName = "raghu.yel002@gmail.com";
	String passWord = "codetheoryio";
	static String email = "user_email";
	static String password = "user_password";
	static String submit = "commit";
	static String multipleDates = "//div[@class='panel-body']//td[1]";
	static String monthlyLink = "//a[text()='Monthly']";
	static String successMsg = "//div[contains(text(),'Signed in successfully.')]";
	static String deleteEntries = "//a[contains(.,'Delete')]";
	static String entrySuccessMsg = "//div[contains(text(),'Entry was successfully created')]";
	static String todaysReadings = "//h3[contains(.,'Daily Report as of')]";
	static String selectingMonthlyReadings = "//a[contains(.,'Monthly')]";
	static String verifyMonth = "//h3[contains(.,'Monthly Report as of')]";
	static String monthlyReadings = "//h3[contains(.,'Monthly Report as of')]";
	static String totalBloodReadings = "//table[@class='table table-condensed']//tbody//td[3]";
	static String highestBloodReading = "//table[@class='table table-condensed']//tbody//td[5]";
	static String lowestBloodReading = "//table[@class='table table-condensed']//tbody//td[4]";
	static String averageBloodReading = "//table[@class='table table-condensed']//tbody//td[6]";
	static String errorMsg = "//div[@class='alert alert-warning fade in']";
	static String infoMsg = "//div[@class='alert alert-info']";
	static String addEntryButton = "//a[contains(text(),'Add new')]";
	static String entryTextBox = "entry_level";
	int actualAverageReading = 0;
	int ExpectedAverageReading = 0;

	@BeforeMethod(alwaysRun = true)
	public void loginUser() {
		driver.get(url);
		driver.manage().window().maximize();
		driver.findElement(By.id(email)).sendKeys(userName);
		driver.findElement(By.id(password)).sendKeys(passWord);
		driver.findElement(By.name(submit)).submit();
		// Verifying user is logged in successfully.
		Assert.assertTrue(driver.findElement(By.xpath(successMsg)).getText().equalsIgnoreCase(successSignInMsg),
				"User is not registerd or entering wrong UserName or password");
		lowestAndHighestReadings();
	}

	@AfterMethod(alwaysRun = true)
	public void deleteBloodReadings() throws InterruptedException {
		navigateToPage(entryPage);

		try {
			wait.until(ExpectedConditions.visibilityOf(driver.findElement(By.xpath(deleteEntries))));
			Thread.sleep(1000);
			while (driver.findElement(By.xpath(deleteEntries)).isDisplayed()) {
				driver.findElement(By.xpath(deleteEntries)).click();
				Alert alt = driver.switchTo().alert();
				alt.accept();
			}
		} catch (NoSuchElementException e) {
			System.out.println(e);
		}
		driver.manage().deleteAllCookies();
		driver.close();
	}

	@Test(description = "Enter the blood values and verify the daily report")
	public void enterBloodEntryAndVerifyDailyReport() throws InterruptedException {
		// Navigate to Level Entries.
		navigateToPage(entryPage);
		// Entering blood values.
		addEntries(bloodGlucoseReadings.length, 0);
		// Verifying success message after entering blood values.
		Assert.assertTrue(driver.findElement(By.xpath(entrySuccessMsg)).getText().equalsIgnoreCase(successEntryMsg),
				"Not able to enter valid blood entries");
		// Navigate to reports.
		navigateToPage(reportPage);
		// Verify by default it is showing same date readings.
		Assert.assertTrue(driver.findElement(By.xpath(todaysReadings)).getText().contains(currentDate),
				"Report not displaying todays readings");
		wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath(totalBloodReadings)));
		// Verify the number of readings per day are same as entered
		List<WebElement> readings = driver.findElements(By.xpath(totalBloodReadings));
		Assert.assertEquals(readings.size(), bloodGlucoseReadings.length, "Number of values not matching");
		highestReadingdaily = Integer.parseInt(driver.findElement(By.xpath(highestBloodReading)).getText().trim());
		lowestReadingdaily = Integer.parseInt(driver.findElement(By.xpath(lowestBloodReading)).getText().trim());
		// Verifying Highest reading
		Assert.assertTrue(highestReadingdaily == highestValue, " Highest values are not matching");
		// Verifying Lowest reading
		Assert.assertTrue(lowestReadingdaily == lowestValue, " Lowest values are not matching");
		actualAverageReading = Integer.parseInt(driver.findElement(By.xpath(averageBloodReading)).getText());
		ExpectedAverageReading = Math.round(totalReadings / bloodGlucoseReadings.length);
		// Verifying average value.
		Assert.assertEquals(actualAverageReading, ExpectedAverageReading, "Average value for day is not mathching");
	}

	@Test(description = "Enter the blood report more than four times and verify the error")
	public void enterBloodValuesMoreThanFourTimesInADay() throws InterruptedException {
		// Navigate to Level Entries.
		driver.get(url + "/entries");
		// Entering blood values.
		addEntries(bloodGlucose1.length, 0);
		// Verify error message saying cannot enter more than 4 times a day
		Assert.assertTrue(driver.findElement(By.xpath("//div[@class='alert alert-warning fade in']")).getText()
				.contains(errorMessage), "Error message is not displaying after  4th entry");
		// verify info message
		Assert.assertTrue(
				driver.findElement(By.xpath("//div[@class='alert alert-info']")).getText().trim().contains(infoMessage),
				"Info message is not displaying after  4th entry");
	}

	@Test(description = "Enter the blood values and verify the monthly report")
	public void enterBloodEntryAndVerifyMonthlyReport() throws InterruptedException {
		// Navigate to Level Entries.
		navigateToPage(entryPage);
		// Entering blood values.
		addEntries(bloodGlucoseReadings.length, 0);
		addEntries(bloodGlucoseReadings.length, -2);
		// Verifying success message after entering blood values.
		Assert.assertTrue(driver.findElement(By.xpath(entrySuccessMsg)).getText().equalsIgnoreCase(successEntryMsg),
				"Not able to enter valid blood entries");
		// Navigate to reports.
		navigateToPage(reportPage);

		// Verify by default it is showing same date readings.
		Assert.assertTrue(driver.findElement(By.xpath(todaysReadings)).getText().contains(currentDate),
				"Report not displaying todays readings");
		driver.findElement(By.xpath(monthlyLink)).click();
		// Verify it is showing both the date readings.
		VerifyDates();
	}

	public void addEntries(int limit, int value) throws InterruptedException {
		for (int i = 0; i < limit; i++) {
			driver.findElement(By.xpath(addEntryButton)).click();
			selectDate_Month_Year(value);
			wait.until(ExpectedConditions.visibilityOfElementLocated(By.id(entryTextBox)));
			driver.findElement(By.id(entryTextBox)).sendKeys("" + bloodGlucoseReadings[i]);
			driver.findElement(By.name(submit)).submit();
		}
	}

	public void navigateToPage(String page) {
		System.out.println(url + page);
		driver.get(url + page);
	}

	public void lowestAndHighestReadings() {
		lowestValue = bloodGlucoseReadings[0];
		highestValue = bloodGlucoseReadings[0];
		for (int i = 0; i < bloodGlucoseReadings.length; i++) {

			if (lowestValue > bloodGlucoseReadings[i]) {
				lowestValue = bloodGlucoseReadings[i];
			}
			if (highestValue < bloodGlucoseReadings[i]) {
				highestValue = bloodGlucoseReadings[i];
			}
			totalReadings = totalReadings + bloodGlucoseReadings[i];
		}
	}

	public void selectDate_Month_Year(int value) throws InterruptedException {
		Select sel;
		Calendar cal = Calendar.getInstance();
		// subtracting a day
		cal.add(Calendar.DATE, value);
		SimpleDateFormat s = new SimpleDateFormat("yyyy-MM-dd");
		currentDate = s.format(new Date());
		ModifiedDate = s.format(new Date(cal.getTimeInMillis()));
		String[] li = ModifiedDate.split("-");
		Thread.sleep(3000);
		// wait.until(ExpectedConditions.visibilityOf(driver.findElement(By.id("entry_recorded_at_3i"))));
		sel = new Select(driver.findElement(By.id("entry_recorded_at_3i")));
		sel.selectByValue("" + Integer.parseInt(li[2]));
		sel = new Select(driver.findElement(By.id("entry_recorded_at_2i")));
		sel.selectByValue("" + Integer.parseInt(li[1]));
		sel = new Select(driver.findElement(By.id("entry_recorded_at_1i")));
		sel.selectByValue("" + Integer.parseInt(li[0]));

	}

	public boolean VerifyDates() {

		int dates = driver.findElements(By.xpath(multipleDates)).size();
		String[] str = new String[dates];
		boolean flag = false;
		boolean flag1 = false;
		List<WebElement> li = driver.findElements(By.xpath(multipleDates));
		for (int i = 0; i < dates; i++) {
			str[i] = li.get(i).getText();
		}
		for (int i = 0; i < str.length; i++) {
			if (currentDate == str[i]) {
				flag = true;
			}
			if (ModifiedDate == str[i]) {
				flag1 = true;
			}
		}
		if (flag1 == flag) {
			return true;
		} else {
			return false;
		}
	}

}
