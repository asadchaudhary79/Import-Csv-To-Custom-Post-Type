<summary>

#ImportCSV

Import Custom Post Type to WordPress from CSV for Free

```php
<?php

function leads_import_csv_shortcode() {
    if (isset($_POST['submit']) && !empty($_FILES['csv_file']['tmp_name'])) {
        $csv_file = $_FILES['csv_file']['tmp_name'];
        $handle = fopen($csv_file, 'r');
        $header = fgetcsv($handle, 1000, ','); // Read and skip the header row

        while (($col = fgetcsv($handle, 1000, ',')) !== FALSE) {
            $post_title = $col[0];
			$phone = $col[1];
            $email = $col[2];
            $lead_source = term_exists($col[3], 'lead-source');
			$author_id = $col[4]; 
			$value = $col[5];
			$organization = $col[6];
			$job_title = $col[7];

            $post_id = wp_insert_post(array(
                'post_title' => $post_title,
                'post_type' => 'leads', //Specifiy your CPT here
                'post_status' => 'publish',
                'post_author' => $author_id,
            ));
			
			if(!empty($post_title))
			{

				if ($post_id) {
					update_post_meta($post_id, 'emailaddress', $email);
					update_post_meta($post_id, 'phonenumber', $phone);
					wp_set_post_terms($post_id, $lead_source, 'lead-source');

					//Assign Agents as relationship
					$relation = jet_engine()->relations->get_active_relations(5);
					$relation->set_update_context('child');
					$relation->update($author_id, $post_id);

					update_post_meta($post_id, 'lead_value', $value);
					update_post_meta($post_id, 'organization', $organization);
					update_post_meta($post_id, 'job-title', $job_title);

					//Set default values for is-deal and lead-status to new
					wp_set_post_terms($post_id, 121, 'is-deal-made');
					wp_set_post_terms($post_id, 101, 'lead-status');

				}
			}
        }

        fclose($handle);
        unlink($csv_file); // Delete the CSV file after import
        echo '<div class="updated"><p>Leads imported successfully!</p></div>';
    }

    ob_start();
    ?>
    <div class="wrap">
        <h6>Import Leads</h6>
        <form method="post" enctype="multipart/form-data" class="flex items-center space-x-4">
            <input type="file" name="csv_file"/>
            <button type="submit" name="submit">
                Import
            </button>
        </form>
    </div>
    <?php
    return ob_get_clean();
}
add_shortcode('import_leads_csv', 'leads_import_csv_shortcode');

```


</details>
